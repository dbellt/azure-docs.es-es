---
title: Clave administrada por el cliente de Azure Monitor
description: Información y pasos para configurar la clave administrada por el cliente para cifrar los datos en las áreas de trabajo de Log Analytics mediante una clave de Azure Key Vault
ms.topic: conceptual
author: yossi-y
ms.author: yossiy
ms.date: 04/21/2021
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: dff35243db327d2b224855e1f65e8e98296d8c8e
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109752272"
---
# <a name="azure-monitor-customer-managed-key"></a>Clave administrada por el cliente de Azure Monitor 

Los datos de Azure Monitor se cifran con claves administradas por Microsoft. Puede usar su propia clave de cifrado para proteger los datos y las consultas guardadas en las áreas de trabajo. Cuando se especifica una clave administrada por el cliente, esa clave se usa para proteger y controlar el acceso a los datos y, una vez configurada, los datos que se envían a las áreas de trabajo se cifran con la clave de Azure Key Vault. Las claves administradas por el cliente ofrecen más flexibilidad para administrar controles de acceso.

Se recomienda revisar la sección [Limitaciones y restricciones](#limitationsandconstraints) que aparece más abajo antes de proceder con la configuración.

## <a name="customer-managed-key-overview"></a>Información general de la clave administrada por el cliente

[El cifrado en reposo](../../security/fundamentals/encryption-atrest.md) es un requisito común de privacidad y seguridad en las organizaciones. Puede dejar que Azure administre por completo el cifrado en reposo, mientras dispone de varias opciones para administrar minuciosamente el cifrado o las claves de cifrado.

Azure Monitor garantiza que todos los datos y las consultas guardadas se cifren en reposo mediante claves administradas por Microsoft (MMK). Azure Monitor también proporciona una opción para el cifrado con su propia clave que se almacena en [Azure Key Vault](../../key-vault/general/overview.md) y le otorga el control para revocar el acceso a los datos en cualquier momento. El uso del cifrado de Azure Monitor es idéntico a la forma en que funciona el [cifrado de Azure Storage](../../storage/common/storage-service-encryption.md#about-azure-storage-encryption).

La clave administrada por el cliente se entrega en [clústeres dedicados](./logs-dedicated-clusters.md) que proporcionan un mayor nivel de protección y control. Los datos ingeridos en clústeres dedicados se cifran dos veces: una en el nivel de servicio mediante claves administradas por Microsoft o claves administradas por el cliente, y otra en el nivel de infraestructura mediante dos algoritmos de cifrado y dos claves diferentes. El [doble cifrado](../../storage/common/storage-service-encryption.md#doubly-encrypt-data-with-infrastructure-encryption) sirve de protección en caso de que uno de los algoritmos o claves de cifrado puedan estar en peligro. En este caso, la capa adicional de cifrado también protege los datos. El clúster dedicado también le permite proteger los datos con un control de [caja de seguridad](#customer-lockbox-preview).

Los datos ingeridos en los últimos 14 días también se conservan en la memoria caché activa (respaldada por SSD) para un funcionamiento eficaz del motor de consultas. Estos datos permanecen cifrados con las claves de Microsoft, con independencia de la configuración de la clave administrada por el cliente, pero el control sobre los datos de SSD se ciñe a la [revocación de claves](#key-revocation). Estamos trabajando para cifrar los datos de SSD con una clave administrada por el cliente en la primera mitad de 2021.

Los clústeres dedicados de Log Analytics usan un [modelo de precios](./logs-dedicated-clusters.md#cluster-pricing-model) de reserva de capacidad que parte de 1000 GB/día.

## <a name="how-customer-managed-key-works-in-azure-monitor"></a>Funcionamiento de las claves administradas por el cliente en Azure Monitor

Azure Monitor usa la identidad administrada para conceder acceso a su instancia de Azure Key Vault. La identidad del clúster de Log Analytics se admite en el nivel de clúster. Para permitir la protección mediante claves administradas por el cliente en varias áreas de trabajo, un nuevo recurso de *clúster* de Log Analytics funciona como una conexión de identidad intermedia entre la instancia de Key Vault y las áreas de trabajo de Log Analytics. El almacenamiento de clúster usa la identidad administrada que \'está asociada al recurso de *clúster* para autenticar el acceso y acceder a Azure Key Vault mediante Azure Active Directory. 

Después de la configuración de la clave administrada por el cliente, los nuevos datos ingeridos en las áreas de trabajo vinculadas asociadas a su clúster dedicado se cifran con la clave. Puede desvincular las áreas de trabajo del clúster en cualquier momento. Después, los nuevos datos se introducen en el almacenamiento de Log Analytics y se cifran con la clave de Microsoft, mientras que puede consultar los datos nuevos y antiguos sin problemas.

> [!IMPORTANT]
> La funcionalidad de la clave administrada por el cliente es regional. La instancia de Azure Key Vault, el clúster y las áreas de trabajo de Log Analytics vinculadas deben estar en la misma región, pero pueden estar en distintas suscripciones.

![Información general de la clave administrada por el cliente](media/customer-managed-keys/cmk-overview.png)

1. Key Vault
2. Recurso *Clúster* de Log Analytics que tiene una identidad administrada con permisos para Key Vault: la identidad se admite en el nivel del almacenamiento de clúster dedicado de Log Analytics
3. Clúster dedicado de Log Analytics
4. Áreas de trabajo vinculadas al recurso de *clúster* 

### <a name="encryption-keys-operation"></a>Operación de claves de cifrado

Hay tres tipos de claves que se usan en el cifrado de datos de Storage:

- **KEK**: clave de cifrado de claves (su clave administrada por el cliente)
- **AEK**: clave de cifrado de la cuenta
- **DEK**: clave de cifrado de datos

Se aplican las reglas siguientes:

- Las cuentas de almacenamiento de clúster de Log Analytics generan una clave de cifrado única para cada cuenta de almacenamiento, lo que se conoce como AEK.
- La AEK se usa para derivar DEK, que son las claves que se usan para cifrar cada bloque de datos escritos en el disco.
- Al configurar la clave en Key Vault y hacer referencia a ella en el clúster, Azure Storage envía solicitudes a Azure Key Vault para ajustar y desajustar AEK para realizar operaciones de cifrado y descifrado de datos.
- Su KEK nunca sale de la instancia de Key Vault.
- Azure Storage usa la identidad administrada que está asociada al recurso *Clúster* para autenticar el acceso y acceder a Azure Key Vault mediante Azure Active Directory.

### <a name="customer-managed-key-provisioning-steps"></a>Pasos de aprovisionamiento de la clave administrada por el cliente

1. Creación de una instancia de Azure Key Vault y almacenamiento de la clave
1. Creación de un clúster
1. Concesión de permisos a la instancia de Key Vault
1. Actualización del clúster con detalles del identificador de clave
1. Vinculación de áreas de trabajo de Log Analytics

La configuración de la clave administrada por el cliente no se admite en Azure Portal actualmente, y el aprovisionamiento se puede realizar mediante [PowerShell](/powershell/module/az.operationalinsights/), la [CLI](/cli/azure/monitor/log-analytics) o solicitudes [REST](/rest/api/loganalytics/).

### <a name="asynchronous-operations-and-status-check"></a>Operaciones asincrónicas y comprobación de estado

Algunos de los pasos de configuración se ejecutan de forma asincrónica porque no se pueden completar rápidamente. El valor de `status` en la respuesta puede ser uno de los siguientes: "EnCurso", "Actualizando", "Eliminando", "Correcto" o "Incorrecto" con el código de error.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

N/D

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

N/D

# <a name="powershell"></a>[PowerShell](#tab/powershell)

N/D

# <a name="rest"></a>[REST](#tab/rest)

Cuando se usa REST, la respuesta devuelve inicialmente un código de estado HTTP 202 (Aceptado) y un encabezado con la propiedad *Azure-AsyncOperation*:
```json
"Azure-AsyncOperation": "https://management.azure.com/subscriptions/subscription-id/providers/Microsoft.OperationalInsights/locations/region-name/operationStatuses/operation-id?api-version=2020-08-01"
```

Puede comprobar el estado de la operación asincrónica mediante el envío de una solicitud GET al punto de conexión del encabezado *Azure-AsyncOperation*:
```rst
GET https://management.azure.com/subscriptions/subscription-id/providers/microsoft.operationalInsights/locations/region-name/operationstatuses/operation-id?api-version=2020-08-01
Authorization: Bearer <token>
```

---

## <a name="storing-encryption-key-kek"></a>Almacenamiento de la clave de cifrado de claves (KEK)

Cree o use una instancia de Azure Key Vault existente en la región en la que se planea el clúster y, a continuación, genere o importe una clave que se usará para el cifrado de registros. Azure Key Vault se debe configurar como recuperable para proteger su clave y el acceso a los datos en Azure Monitor. Puede comprobar esta configuración en las propiedades de Key Vault: tanto la *Eliminación temporal* como la *Protección de purga* deben estar habilitadas.

![Configuración de la eliminación temporal y la protección de purga](media/customer-managed-keys/soft-purge-protection.png)

Esta configuración puede actualizarse en Key Vault a través de la CLI y PowerShell:

- [eliminación temporal](../../key-vault/general/soft-delete-overview.md)
- La [Protección de purga](../../key-vault/general/soft-delete-overview.md#purge-protection) protege contra la eliminación forzada del secreto o almacén incluso después de la eliminación temporal.

## <a name="create-cluster"></a>Crear clúster

Los clústeres admiten dos [tipos de identidad administrada](../../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types): Asignado por el sistema y asignado por el usuario, mientras que una sola identidad se puede definir en un clúster en función del escenario. 
- La identidad administrada asignada por el sistema es más sencilla y se genera automáticamente con la creación del clúster cuando la identidad `type` se establece en "*SystemAssigned*". Esta identidad se puede usar más adelante para conceder acceso de almacenamiento a su instancia de Key Vault para las operaciones de encapsulado y desencapsulado. 
  
  Configuración de identidad en el clúster para la identidad administrada asignada por el sistema
  ```json
  {
    "identity": {
      "type": "SystemAssigned"
      }
  }
  ```

- Si desea configurar una clave administrada por el cliente en la creación del clúster, debe tener una clave y una identidad asignada por el usuario concedida de antemano en su instancia de Key Vault y, a continuación, crear el clúster con estos valores: la identidad `type` establecida como "*UserAssigned*", `UserAssignedIdentities` con el *identificador de recurso* de la identidad.

  Configuración de identidad en el clúster para la identidad administrada asignada por el usuario
  ```json
  {
  "identity": {
  "type": "UserAssigned",
    "userAssignedIdentities": {
      "subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.ManagedIdentity/UserAssignedIdentities/<cluster-assigned-managed-identity>"
      }
  }
  ```

> [!IMPORTANT]
> No se puede usar una identidad administrada asignada por el usuario si su instancia de Key Vault está en un vínculo privado (vNet). En este escenario puede usar la identidad administrada asignada por el sistema.

Siga el procedimiento que se muestra en el [artículo Clústeres dedicados](./logs-dedicated-clusters.md#creating-a-cluster). 

## <a name="grant-key-vault-permissions"></a>Concesión de permisos a Key Vault

Cree una directiva de acceso en Key Vault para conceder permisos al clúster. Estos permisos los utiliza el almacenamiento de Azure Monitor subyacente. Abra su instancia de Key Vault en Azure Portal y haga clic en *"Directivas de acceso"* y luego en *"+ Agregar directiva de acceso"* para crear una directiva con esta configuración:

- Permisos de clave: seleccione *"Obtener"* , *"Encapsular clave"* y *"Desencapsular clave"* .
- Seleccionar la entidad de seguridad: en función del tipo de identidad utilizado en el clúster (identidad administrada asignada por el sistema o por el usuario), escriba el nombre del clúster o el identificador de la entidad de seguridad del clúster para el nombre de la identidad administrada asignada por el sistema o de la identidad administrada asignada por el usuario.

![Concesión de permisos a Key Vault](media/customer-managed-keys/grant-key-vault-permissions-8bit.png)

El permiso *Obtener* es necesario para comprobar que su instancia de Azure Key Vault está configurada como recuperable con el fin de proteger su clave y el acceso a sus datos de Azure Monitor.

## <a name="update-cluster-with-key-identifier-details"></a>Actualización del clúster con detalles del identificador de clave

Todas las operaciones del clúster requieren el permiso de acción `Microsoft.OperationalInsights/clusters/write`. Este permiso se puede conceder a través del rol Propietario o Colaborador que contiene la acción `*/write` o a través del rol Colaborador de Log Analytics que contiene la acción `Microsoft.OperationalInsights/*`.

En este paso se actualiza el almacenamiento de Azure Monitor con la clave y la versión que se van a usar para el cifrado de los datos. Cuando se actualiza, la clave nueva se usa para ajustar y desajustar la clave de almacenamiento (AEK).

>[!IMPORTANT]
>- La rotación de claves puede ser automática o requerir una actualización de clave explícita. Consulte [Rotación de claves](#key-rotation) para determinar la estrategia adecuada antes de actualizar los detalles del identificador de clave del clúster.
>- La actualización del clúster no debe incluir los detalles de identidad y de identificador de clave en la misma operación. Si necesita actualizar ambos valores, la actualización debe realizarse en dos operaciones consecutivas.

![Concesión de permisos a Key Vault](media/customer-managed-keys/key-identifier-8bit.png)

Actualice KeyVaultProperties en el clúster con los detalles del identificador de clave.

Esta operación es asincrónica y puede tardar un tiempo en completarse.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

N/D

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az monitor log-analytics cluster update --name "cluster-name" --resource-group "resource-group-name" --key-name "key-name" --key-vault-uri "key-uri" --key-version "key-version"
```
# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
Update-AzOperationalInsightsCluster -ResourceGroupName "resource-group-name" -ClusterName "cluster-name" -KeyVaultUri "key-uri" -KeyName "key-name" -KeyVersion "key-version"
```

# <a name="rest"></a>[REST](#tab/rest)

```rst
PATCH https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.OperationalInsights/clusters/cluster-name?api-version=2020-08-01
Authorization: Bearer <token> 
Content-type: application/json
 
{
  "properties": {
    "keyVaultProperties": {
      "keyVaultUri": "https://key-vault-name.vault.azure.net",
      "kyName": "key-name",
      "keyVersion": "current-version"
  },
  "sku": {
    "name": "CapacityReservation",
    "capacity": 1000
  }
}
```

**Respuesta**

La propagación de la clave tarda unos minutos en completarse. Puede comprobar el estado de actualización de dos maneras:
1. Copie el valor de la dirección URL de Azure-AsyncOperation de la respuesta y siga la [comprobación del estado de operaciones asincrónicas](#asynchronous-operations-and-status-check).
2. Envíe una solicitud GET en el clúster y examine las propiedades de *KeyVaultProperties*. La clave actualizada recientemente se debe devolver en la respuesta.

Una respuesta a la solicitud GET debe ser similar a la siguiente cuando se complete la actualización de la clave: 202 (Aceptado) y encabezado
```json
{
  "identity": {
    "type": "SystemAssigned",
    "tenantId": "tenant-id",
    "principalId": "principle-id"
    },
  "sku": {
    "name": "capacityReservation",
    "capacity": 1000,
    "lastSkuUpdate": "Sun, 22 Mar 2020 15:39:29 GMT"
    },
  "properties": {
    "keyVaultProperties": {
      "keyVaultUri": "https://key-vault-name.vault.azure.net",
      "kyName": "key-name",
      "keyVersion": "current-version"
      },
    "provisioningState": "Succeeded",
    "billingType": "cluster",
    "clusterId": "cluster-id"
  },
  "id": "/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.OperationalInsights/clusters/cluster-name",
  "name": "cluster-name",
  "type": "Microsoft.OperationalInsights/clusters",
  "location": "region-name"
}
```

---

## <a name="link-workspace-to-cluster"></a>Vinculación del área de trabajo al clúster

> [!IMPORTANT]
> Este paso solo debe realizarse una vez completada la Log Analytics el aprovisionamiento del clúster. Si vincula áreas de trabajo e ingiere datos antes del aprovisionamiento, los datos ingeridos se quitarán y no se podrán recuperar.

Debe tener permisos de "escritura" en el área de trabajo y en el clúster para realizar esta operación, que incluye `Microsoft.OperationalInsights/workspaces/write` y `Microsoft.OperationalInsights/clusters/write`.

Siga el procedimiento que se muestra en el [artículo Clústeres dedicados](./logs-dedicated-clusters.md#link-a-workspace-to-cluster).

## <a name="key-revocation"></a>Revocación de claves

> [!IMPORTANT]
> - La manera recomendada para revocar el acceso a los datos es deshabilitar la clave o eliminar la directiva de acceso de su instancia de Key Vault.
> - Al establecer `identity` `type` del clúster en "None", también se revoca el acceso a los datos, pero no se recomienda este enfoque, ya que no se puede revertir la revocación al redefinir `identity` en el clúster sin abrir la solicitud de soporte técnico.

El almacenamiento de clúster siempre respetará los cambios en los permisos de las claves en el plazo de una hora (normalmente antes), y Storage dejará de estar disponible. Los datos nuevos que se ingieren en áreas de trabajo vinculadas al clúster se quitan y no se podrán recuperar, los datos dejan de estar accesibles y las consultas a estas áreas de trabajo generan un error. Los datos ingeridos anteriormente permanecerán en el almacenamiento siempre que se no se eliminen el clúster ni las áreas de trabajo. Los datos inaccesibles se rigen por la directiva de retención de datos y se purgarán cuando se alcance la retención. Los datos ingeridos en los últimos 14 días también se conservan en la memoria caché activa (respaldada por SSD) para un funcionamiento eficaz del motor de consultas. Esto se elimina en la operación de revocación de claves y se convierte en inaccesible.

El almacenamiento del clúster comprueba periódicamente su instancia de Key Vault para intentar desencapsular la clave de cifrado y, una vez haya obtenido el acceso, procederá con la ingesta de datos y la reanudación de la consulta en un plazo de 30 minutos.

## <a name="key-rotation"></a>Rotación de claves

La rotación de clave tiene dos modos: 
- Rotación automática: cuando el clúster se actualiza con ```"keyVaultProperties"``` pero se omite la propiedad ```"keyVersion"```, o bien se establece en ```""```, el almacenamiento usará las últimas versiones de forma automática.
- Actualización de la versión de clave explícita: cuando el clúster se actualiza y se proporciona la versión de la clave en la propiedad ```"keyVersion"```, las nuevas versiones de clave requieren una actualización ```"keyVaultProperties"``` explícita en el clúster. Consulte [Actualización del clúster con detalles del identificador de clave](#update-cluster-with-key-identifier-details). Si genera la nueva versión de la clave en la instancia de Key Vault, pero no la actualiza en el clúster, el almacenamiento de clúster de Log Analytics seguirá usando la clave anterior. Si deshabilita o elimina la clave anterior antes de actualizar la nueva clave en el clúster, obtendrá el estado de [revocación de clave](#key-revocation).

Se puede acceder a todos los datos después de la operación de rotación de claves, incluidos los datos ingeridos antes y después de la rotación, ya que todos los datos permanecen cifrados mediante la clave de cifrado de cuenta (AEK), mientras que la AEK ahora se cifra con la nueva versión de la clave de cifrado de claves (KEK).

## <a name="customer-managed-key-for-saved-queries-and-log-alerts"></a>Clave administrada por el cliente para consultas guardadas y alertas de registro

El lenguaje de consulta utilizado en Log Analytics es expresivo y puede contener información confidencial en los comentarios que se agregan a las consultas o en la sintaxis de la consulta. Algunas organizaciones requieren que dicha información se mantenga protegida en el marco de la directiva de la clave administrada por el cliente y debe guardar las consultas cifradas con su clave. Azure Monitor le permite almacenar consultas de *búsquedas guardadas* y de *alertas del registro* cifradas con su clave en su propia cuenta de almacenamiento cuando se conecta al área de trabajo. 

> [!NOTE]
> Las consultas de Log Analytics se pueden guardar en varios almacenes según el escenario usado. Las consultas permanecen cifradas con la clave de Microsoft (MMK) en los escenarios siguientes, con independencia de la configuración de la clave administrada por el cliente: Libros en Azure Monitor, paneles de Azure, Azure Logic Apps, Azure Notebooks y runbooks de automatización.

Cuando traiga su propio almacenamiento (BYOS) y lo vincule a su área de trabajo, el servicio carga consultas de *búsquedas guardadas* y de *alertas del registro* en la cuenta de almacenamiento. Esto significa que puede controlar la cuenta de almacenamiento y la [directiva de cifrado en reposo](../../storage/common/customer-managed-keys-overview.md) con la misma clave que se usa para cifrar los datos en el clúster de Log Analytics o con una clave diferente. Sin embargo, será responsable de los costos asociados a esa cuenta de almacenamiento. 

**Consideraciones antes de establecer la clave administrada por el cliente en las consultas**
* Debe tener permisos de "escritura" en el área de trabajo y la cuenta de almacenamiento.
* Asegúrese de crear la cuenta de almacenamiento en la misma región en la que se encuentra el área de trabajo de Log Analytics.
* Las *búsquedas guardadas* en el almacenamiento se consideran artefactos de servicio y su formato puede cambiar.
* Las *búsquedas guardadas* existentes se eliminan del área de trabajo. Copie las *búsquedas guardadas* que necesite antes de la configuración. Puede ver las *búsquedas guardadas* mediante [PowerShell](/powershell/module/az.operationalinsights/get-azoperationalinsightssavedsearch).
* No se admite el historial de consultas y no podrá ver las consultas que se han ejecutado.
* Puede vincular una sola cuenta de almacenamiento al área de trabajo con el fin de guardar las consultas, pero se puede usar tanto para las consultas de *búsquedas guardadas* como para las de *alertas del registro*.
* No se admite el anclaje a un panel.
* Las alertas del registro desencadenadas no contendrán resultados de búsqueda ni consultas de alertas. Puede usar [dimensiones de alerta](../alerts/alerts-unified-log.md#split-by-alert-dimensions) para obtener contexto en las alertas desencadenadas.

**Configuración de BYOS para las consultas de búsquedas guardadas**

Vincule una cuenta de almacenamiento de *Consulta* con el área de trabajo. Las consultas de *búsquedas guardadas* se guardan en la cuenta de almacenamiento. 

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

N/D

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
$storageAccountId = '/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<storage name>'
az monitor log-analytics workspace linked-storage create --type Query --resource-group "resource-group-name" --workspace-name "workspace-name" --storage-accounts $storageAccountId
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
$storageAccount.Id = Get-AzStorageAccount -ResourceGroupName "resource-group-name" -Name "storage-account-name"
New-AzOperationalInsightsLinkedStorageAccount -ResourceGroupName "resource-group-name" -WorkspaceName "workspace-name" -DataSourceType Query -StorageAccountIds $storageAccount.Id
```

# <a name="rest"></a>[REST](#tab/rest)

```rst
PUT https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.OperationalInsights/workspaces/<workspace-name>/linkedStorageAccounts/Query?api-version=2020-08-01
Authorization: Bearer <token> 
Content-type: application/json
 
{
  "properties": {
    "dataSourceType": "Query", 
    "storageAccountIds": 
    [
      "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<storage-account-name>"
    ]
  }
}
```

---

Después de la configuración, se guardará en el almacenamiento cualquier nueva consulta de *búsqueda guardada*.

**Configuración de BYOS para las consultas de alertas del registro**

Vincule una cuenta de almacenamiento de *Alertas* al área de trabajo. Las consultas de *alertas del registro* se guardan en la cuenta de almacenamiento. 

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

N/D

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
$storageAccountId = '/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<storage name>'
az monitor log-analytics workspace linked-storage create --type ALerts --resource-group "resource-group-name" --workspace-name "workspace-name" --storage-accounts $storageAccountId
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
$storageAccount.Id = Get-AzStorageAccount -ResourceGroupName "resource-group-name" -Name "storage-account-name"
New-AzOperationalInsightsLinkedStorageAccount -ResourceGroupName "resource-group-name" -WorkspaceName "workspace-name" -DataSourceType Alerts -StorageAccountIds $storageAccount.Id
```

# <a name="rest"></a>[REST](#tab/rest)

```rst
PUT https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.OperationalInsights/workspaces/<workspace-name>/linkedStorageAccounts/Alerts?api-version=2020-08-01
Authorization: Bearer <token> 
Content-type: application/json
 
{
  "properties": {
    "dataSourceType": "Alerts", 
    "storageAccountIds": 
    [
      "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<storage-account-name>"
    ]
  }
}
```

---

Después de la configuración, se guardará en el almacenamiento cualquier nueva consulta de alertas.

## <a name="customer-lockbox-preview"></a>Caja de seguridad del cliente (versión preliminar)

Caja de seguridad le proporciona el control para aprobar o rechazar la solicitud de ingeniero de Microsoft para acceder a los datos durante una solicitud de soporte técnico.

En Azure Monitor, tiene este control sobre los datos en las áreas de trabajo vinculadas al clúster dedicado de Log Analytics. El control Caja de seguridad se aplica a los datos almacenados en un clúster dedicado de Log Analytics en el que se mantiene aislado en las cuentas de almacenamiento del clúster en su suscripción protegida de Caja de seguridad.  

Más información sobre [Caja de seguridad del cliente de Microsoft Azure](../../security/fundamentals/customer-lockbox-overview.md)

## <a name="customer-managed-key-operations"></a>Operaciones de la clave administrada por el cliente

La clave administrada por el cliente se proporciona en un clúster dedicado, y se hace referencia a estas operaciones en el [artículo sobre el clúster dedicado](./logs-dedicated-clusters.md#change-cluster-properties).

- Obtención de todos los clústeres de un grupo de recursos  
- Obtención de todos los clústeres de una suscripción
- Actualización de la *reserva de capacidad* en el clúster
- Actualización de *billingType* en el clúster
- Desvinculación de un área de trabajo de un clúster
- Eliminación de clúster

## <a name="limitations-and-constraints"></a>Limitaciones y restricciones

- El número máximo de clústeres por región y suscripción es 2.

- El número máximo de áreas de trabajo que se pueden vincular a un clúster es 1000.

- Puede vincular un área de trabajo al clúster y desvincularla después. El número de operaciones de vinculación de área de trabajo en un área de trabajo determinada en un período de 30 días se limita a 2.

- El cifrado de la clave administrada por el cliente se aplica a los datos recién ingeridos después del tiempo de configuración. Los datos que se ingieren antes de la configuración, permanecen cifrados con la clave de Microsoft. Puede consultar fácilmente los datos ingeridos antes y después de la configuración de la clave administrada por el cliente.

- La instancia de Azure Key Vault debe configurarse como recuperable. Las siguientes propiedades no están habilitadas de forma predeterminada y deben configurarse mediante la CLI o PowerShell:<br>
  - [eliminación temporal](../../key-vault/general/soft-delete-overview.md)
  - La [protección de purgas](../../key-vault/general/soft-delete-overview.md#purge-protection) debe estar activada si quiere tener protección frente a posibles eliminaciones forzadas de secretos o del almacén, incluso después de su eliminación temporal.

- Actualmente no se admite el traslado de un clúster a otro grupo de recursos o a otra suscripción.

- Su instancia de Azure Key Vault, el clúster y las áreas de trabajo deben estar en la misma región y en el mismo inquilino de Azure Active Directory (Azure AD), pero pueden estar en distintas suscripciones.

- La actualización del clúster no debe incluir los detalles de identidad y de identificador de clave en la misma operación. En caso de que deba actualizar ambos valores, la actualización debe realizarse en dos operaciones consecutivas.

- La caja de seguridad no está disponible actualmente en China. 

- El [cifrado doble](../../storage/common/storage-service-encryption.md#doubly-encrypt-data-with-infrastructure-encryption) se configura automáticamente para los clústeres creados a partir de octubre de 2020 en las regiones compatibles. Puede comprobar si el clúster está configurado para el cifrado doble mediante el envío de una solicitud GET en el clúster y observando que el valor `isDoubleEncryptionEnabled` sea `true` para los clústeres con el cifrado doble habilitado. 
  - Si crea un clúster y recibe un error que dice que la región no admite el cifrado doble para clústeres, puede crear el clúster sin cifrado doble agregando `"properties": {"isDoubleEncryptionEnabled": false}` en el cuerpo de la solicitud REST.
  - La configuración de cifrado doble no se puede cambiar después de crear el clúster.

  - Si el clúster se establece con la identidad administrada asignada por el usuario, el establecimiento de `UserAssignedIdentities` con `None` suspende el clúster e impide el acceso a los datos, pero no se puede revertir la revocación ni activar el clúster sin abrir la solicitud de soporte técnico. Esta limitación no se aplica a la identidad administrada asignada por el sistema.

  - No se puede usar una clave administrada por el cliente con una identidad administrada asignada por el usuario si su instancia de Key Vault está en un vínculo privado (vNet). En este escenario puede usar la identidad administrada asignada por el sistema.

## <a name="troubleshooting"></a>Solución de problemas

- Comportamiento con la disponibilidad de Key Vault
  - En el funcionamiento normal, Storage almacena en caché la AEK durante breves períodos de tiempo y vuelve regularmente a Key Vault para desajustarla.
    
  - En el caso de los errores de conexión transitorios (tiempos de espera, errores de conexión, problemas de DNS), Storage permite que las claves permanezcan en la memoria caché durante un período de tiempo más prolongado; así es más fácil resolver pequeños problemas de disponibilidad. Las funciones de consulta e ingesta continúan sin interrupción.
    
  - La falta de disponibilidad de un sitio activo durante unos 30 minutos hará que la cuenta de Storage deje de estar disponible. La función de consulta no está disponible y los datos ingeridos se copian en caché durante varias horas mediante la clave de Microsoft para evitar la pérdida de datos. Cuando se restaura el acceso a Key Vault, la consulta está disponible y los datos almacenados en caché temporalmente se ingieren en el almacén de datos y se cifran con la clave administrada por el cliente.

  - Frecuencia de acceso a Key Vault: la frecuencia con la que Azure Monitor Storage accede a Key Vault para las operaciones de encapsulado y desencapsulado es de entre 6 y 60 segundos.

- Si actualiza el clúster mientras este está en el estado de aprovisionamiento o actualización, se producirá un error en la actualización.

- Si se produce un error de conflicto al crear un clúster, es posible que haya eliminado el clúster en los últimos 14 días y que esté en un período de eliminación temporal. El nombre del clúster permanece reservado durante el período de eliminación temporal y no se puede crear un nuevo clúster con ese nombre. El nombre se libera después del período de eliminación temporal cuando el clúster se elimina de forma permanente.

- Se producirá un error en la vinculación del área de trabajo al clúster si está vinculada a otro clúster.

- Si crea un clúster y especifica el valor de KeyVaultProperties inmediatamente, puede producirse un error en la operación, ya que la directiva de acceso no se puede definir hasta que se asigne la identidad del sistema al clúster.

- Si actualiza un clúster existente con KeyVaultProperties y la directiva de acceso de clave "Get" no se encuentra en Key Vault, se producirá un error en la operación.

- Si no puede implementar el clúster, compruebe que la instancia de Azure Key Vault, el clúster y las áreas de trabajo de Log Analytics vinculadas se encuentran en la misma región. Puede estar en distintas suscripciones.

- Si actualiza la versión de la clave en Key Vault y no actualiza los nuevos detalles del identificador de clave en el clúster, el clúster de Log Analytics seguirá usando la clave anterior y los datos serán inaccesibles. Actualice los nuevos detalles de identificador de clave del clúster para reanudar la ingesta de datos y la capacidad de consultar los datos.

- Algunas operaciones son largas y pueden tardar un tiempo en completarse. Estas operaciones son las de creación de clúster, actualización de la clave de clúster y eliminación de clúster. Puede comprobar el estado de la operación de dos maneras:
  1. Cuando se utiliza REST, copie el valor de la dirección URL de Azure-AsyncOperation de la respuesta y siga la [comprobación del estado de operaciones asincrónicas](#asynchronous-operations-and-status-check).
  2. Envíe una solicitud GET al clúster o al área de trabajo y observe la respuesta. Por ejemplo, un área de trabajo desvinculada no tendrá el elemento *clusterResourceId* en la sección *features*.

- Mensajes de error
  
  **Creación de un clúster**
  -  400: El nombre del clúster no es válido. El nombre del clúster puede contener los caracteres a-z, A-Z, 0-9 y una longitud de 3 a 63.
  -  400: El cuerpo de la solicitud es NULL o tiene un formato incorrecto.
  -  400: El nombre de SKU no es válido. Establezca el nombre de SKU en capacityReservation.
  -  400: Se proporcionó Capacity, pero la SKU no es capacityReservation. Establezca el nombre de la SKU en capacityReservation.
  -  400: Falta Capacity en la SKU. Establezca el valor de Capacity en 1000 o más en incrementos de 100 (GB).
  -  400: Capacity en la SKU no está en el rango. Debe ser 1000 como mínimo y hasta la capacidad máxima permitida que está disponible en "Uso y costos estimados" en el área de trabajo.
  -  400: Capacity está bloqueado durante 30 días. Se permite la reducción de la capacidad 30 días después de la actualización.
  -  400: No se estableció ninguna SKU. Establezca el nombre de la SKU en capacityReservation y el valor de Capacity en 1000 o más en incrementos de 100 (GB).
  -  400: Identity es NULL o está vacío. Establezca Identity en el tipo systemAssigned.
  -  400: KeyVaultProperties se estableció en la creación. Actualice KeyVaultProperties después de la creación del clúster.
  -  400: No se puede ejecutar la operación ahora. La operación asincrónica está en un estado distinto del correcto. El clúster debe completar su operación antes de realizar cualquier operación de actualización.

  **Actualización del clúster:**
  -  400: El clúster está en estado de eliminación. La operación asincrónica está en curso. El clúster debe completar su operación antes de realizar cualquier operación de actualización.
  -  400: KeyVaultProperties no está vacío, pero tiene un formato incorrecto. Consulte [actualización de identificador de clave](#update-cluster-with-key-identifier-details).
  -  400: No se pudo validar la clave en Key Vault. Podría deberse a la falta de permisos o a que la clave no existe. Verifique que [estableció la clave y la directiva de acceso](#grant-key-vault-permissions) en Key Vault.
  -  400: No se puede recuperar la clave. Key Vault debe establecerse para la eliminación temporal y protección de purga. Consulte la [documentación de Key Vault](../../key-vault/general/soft-delete-overview.md).
  -  400: No se puede ejecutar la operación ahora. Espere a que se complete la operación asincrónica e inténtelo de nuevo.
  -  400: El clúster está en estado de eliminación. Espere a que se complete la operación asincrónica e inténtelo de nuevo.

  **Obtención del clúster**
    -  404: No se encontró el clúster, es posible que se haya eliminado. Si intenta crear un clúster con ese nombre y obtiene un conflicto, el clúster se encuentra en eliminación temporal durante 14 días. Puede ponerse en contacto con soporte técnico para recuperarlo o usar otro nombre para crear un nuevo clúster. 

  **Eliminación del clúster:**
    -  409: No se puede eliminar un clúster mientras está en estado de aprovisionamiento. Espere a que se complete la operación asincrónica e inténtelo de nuevo.

  **Vinculación del área de trabajo**
  -  404: No se encuentra el área de trabajo. El área de trabajo que especificó no existe o se eliminó.
  -  409: Operación en curso de vinculación o desvinculación del área de trabajo.
  -  400: No se encontró el clúster, el clúster que especificó no existe o se eliminó. Si intenta crear un clúster con ese nombre y obtiene un conflicto, el clúster se encuentra en eliminación temporal durante 14 días. Puede ponerse en contacto con el soporte técnico para recuperarlo.

  **Desvinculación del área de trabajo**
  -  404: No se encuentra el área de trabajo. El área de trabajo que especificó no existe o se eliminó.
  -  409: Operación en curso de vinculación o desvinculación del área de trabajo.
## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre la [facturación de clústeres dedicados de Log Analytics](./manage-cost-storage.md#log-analytics-dedicated-clusters)
- Obtenga más información sobre el [diseño adecuado de áreas de trabajo de Log Analytics](./design-logs-deployment.md)
