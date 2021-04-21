---
title: 'Tutorial: Uso de identidades administradas para acceder a Azure Cosmos DB (Windows): Azure AD'
description: Este tutorial le guía por el proceso de usar una identidad administrada asignada por el sistema en una máquina virtual Windows para acceder a Azure Cosmos DB.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/10/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 15deae3de20de579bff880a6cb7c9e44719a63ed
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107776436"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-cosmos-db"></a>Tutorial: Uso de las identidades administradas asignadas por el sistema de una máquina virtual Windows para acceder a Azure Cosmos DB

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

En este tutorial se muestra cómo usar una identidad administrada asignada por el sistema en una máquina virtual Windows para acceder a Cosmos DB. Aprenderá a:

> [!div class="checklist"]
> * Creación de una cuenta de Cosmos DB
> * Conceder acceso a la identidad administrada asignada por el sistema en la máquina virtual Windows a las claves de acceso de la cuenta de Cosmos DB
> * Obtener un token de acceso mediante una identidad administrada asignada por el sistema de la máquina virtual Windows para llamar a Azure Resource Manager
> * Obtención de las claves de acceso desde Azure Resource Manager para realizar llamadas a Cosmos DB

## <a name="prerequisites"></a>Prerrequisitos

- Si no está familiarizado con la característica Managed Identities for Azure Resources, consulte esta [introducción](overview.md). 
- Si no tiene una cuenta de Azure, [regístrese para obtener una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.
- Para realizar la creación de recursos necesarios y la administración de roles, la cuenta debe tener permisos de "Propietario" en el ámbito adecuado (en su suscripción o en un grupo de recursos). Si necesita ayuda con la asignación de roles, consulte [Asignación de roles de Azure para administrar el acceso a los recursos de una suscripción de Azure](../../role-based-access-control/role-assignments-portal.md).
- [Instalar la versión más reciente de Azure PowerShell](/powershell/azure/install-az-ps)
- También necesita una máquina virtual Windows que tenga habilitadas las identidades administradas asignadas por el sistema.
  - Si necesita crear una máquina virtual para este tutorial, puede seguir el artículo titulado [Configurar identidades administradas para recursos de Azure en una VM mediante Azure Portal](./qs-configure-portal-windows-vm.md#system-assigned-managed-identity).

## <a name="create-a-cosmos-db-account"></a>Creación de una cuenta de Cosmos DB 

Si aún no tiene una, cree una cuenta de Cosmos DB. También puede omitir este paso y usar una cuenta de Cosmos DB existente. 

1. Haga clic en el botón **+Crear un recurso** de la esquina superior izquierda de Azure Portal.
2. Haga clic en **Bases de datos** y, a continuación, en **Azure Cosmos DB** para mostrar el panel "Nueva cuenta".
3. Escriba un **identificador** para la cuenta de Cosmos DB, el cual se utilizará más adelante.  
4. **API** se debe establecer en "SQL". El enfoque descrito en este tutorial se puede utilizar con los otros tipos de API disponibles, pero los pasos de este tutorial son para la API de SQL.
5. Asegúrese de que **Suscripción** y **Grupo de recursos** coinciden con los que especificó cuando creó la máquina virtual en el paso anterior.  Seleccione una **Ubicación** en la que Cosmos DB esté disponible.
6. Haga clic en **Crear**.

### <a name="create-a-collection"></a>Creación de una colección 

A continuación, agregue una colección de datos en la cuenta de Cosmos DB que podrá consultar en pasos posteriores.

1. Vaya a la cuenta de Cosmos DB recién creada.
2. En la pestaña **Información general**, haga clic en el botón **+/Agregar colección** y aparecerá un panel "Agregar colección".
3. Proporcione para la colección un identificador de base de datos, el identificador de la colección, seleccione una capacidad de almacenamiento, escriba una clave de partición, escriba un valor de rendimiento y, luego, haga clic en **Aceptar**.  Para este tutorial, es suficiente con utilizar "Test" como identificador de la base de datos e identificador de la colección, seleccionar una capacidad de almacenamiento fijo y el rendimiento más bajo (400 RU/s).  


## <a name="grant-access"></a>Conceder acceso

En esta sección se muestra cómo conceder acceso a la identidad administrada asignada por el sistema en la máquina virtual Windows a las claves de acceso de la cuenta de Cosmos DB. Cosmos DB no admite la autenticación de Azure AD de forma nativa. No obstante, puede usar una identidad administrada asignada por el sistema para recuperar una clave de acceso de Cosmos DB desde Resource Manager y usar dicha clave para acceder a Cosmos DB. En este paso, va a conceder a la identidad administrada asignada por el sistema de la máquina virtual Windows acceso a las claves de la cuenta de Cosmos DB.

Para conceder a la identidad administrada asignada por el sistema de la máquina virtual Windows acceso a la cuenta de Cosmos DB en Azure Resource Manager mediante PowerShell, actualice los siguientes valores:

- `<SUBSCRIPTION ID>`
- `<RESOURCE GROUP>`
- `<COSMOS DB ACCOUNT NAME>`

Cosmos DB admite dos niveles de granularidad en las claves de acceso: acceso de lectura y escritura a la cuenta y acceso de solo lectura a la cuenta.  Asigne el rol `DocumentDB Account Contributor` si desea obtener claves de lectura y escritura para la cuenta o bien asigne el rol `Cosmos DB Account Reader Role` si desea obtener claves de solo lectura para la cuenta.  Para este tutorial, asigne `Cosmos DB Account Reader Role`:

```azurepowershell
$spID = (Get-AzVM -ResourceGroupName myRG -Name myVM).identity.principalid
New-AzRoleAssignment -ObjectId $spID -RoleDefinitionName "Cosmos DB Account Reader Role" -Scope "/subscriptions/<mySubscriptionID>/resourceGroups/<myResourceGroup>/providers/Microsoft.DocumentDb/databaseAccounts/<COSMOS DB ACCOUNT NAME>"
```

>[!NOTE]
> Tenga en cuenta que si no puede realizar una operación, es posible que no tenga los permisos adecuados. Si desea tener acceso de escritura a las claves, debe usar un rol de Azure como Colaborador de la cuenta de DocumentDB o crear un rol personalizado. Para más información, consulte [Control de acceso basado en rol en Azure Cosmos DB](../../cosmos-db/role-based-access-control.md).

## <a name="access-data"></a>Acceso a los datos

En esta sección se muestra cómo llamar a Azure Resource Manager con un token de acceso para la identidad administrada asignada por el sistema de la máquina virtual Windows. En el resto del tutorial, vamos a trabajar desde la máquina virtual que se creó anteriormente. 

Tiene que instalar la versión más reciente de la [CLI de Azure](/cli/azure/install-azure-cli) en la máquina virtual Windows.

### <a name="get-an-access-token"></a>Obtención de un token de acceso

1. En Azure Portal, vaya a **Máquinas virtuales**, vaya a la máquina virtual Windows y, a continuación, desde la página **Información general**, haga clic en **Conectar** en la parte superior. 
2. Escriba su **nombre de usuario** y **contraseña** que agregó cuando creó la máquina virtual Windows. 
3. Ahora que ha creado una **conexión a Escritorio remoto** con la máquina virtual, abra PowerShell en la sesión remota.
4. Mediante el comando Invoke-WebRequest de Powershell, realice una solicitud al punto de conexión local de Managed Identities for Azure Resources y obtenga un token de acceso para Azure Resource Manager.

   ```powershell
   $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -Method GET -Headers @{Metadata="true"}
   ```

   > [!NOTE]
   > El valor del parámetro "resource" debe coincidir exactamente con el que Azure AD espera. Al usar el id. de recurso de Azure Resource Manager, debe incluir la barra diagonal final en el URI.
    
   A continuación, extraiga el elemento "Content", que se almacena como una cadena con formato de notación de objetos JavaScript (JSON) en el objeto $response. 
    
   ```powershell
   $content = $response.Content | ConvertFrom-Json
   ```
   Luego, extraiga el token de acceso de la respuesta.
    
   ```powershell
   $ArmToken = $content.access_token
   ```

### <a name="get-access-keys"></a>Obtención de las claves de acceso 

En esta sección se muestra cómo obtener las claves de acceso desde Azure Resource Manager para realizar llamadas a Cosmos DB. Vamos a usar PowerShell para llamar a Resource Manager mediante el token de acceso que obtuvimos anteriormente para recuperar la clave de acceso de la cuenta de Cosmos DB. Una vez que tenemos la clave de acceso, podemos realizar consultas en Cosmos DB. Use sus propios valores para reemplazar las entradas siguientes:

- `<SUBSCRIPTION ID>`
- `<RESOURCE GROUP>`
- `<COSMOS DB ACCOUNT NAME>` 
- Reemplace el valor de `<ACCESS TOKEN>` por el token de acceso que se recuperó anteriormente. 

>[!NOTE]
>Si quiere recuperar claves de lectura y escritura, use el tipo de operación de claves `listKeys`.  Si quiere recuperar las claves de solo lectura, use el tipo de operación de claves `readonlykeys`. Si no puede usar "listkeys", compruebe que ha asignado el [rol adecuado](../../role-based-access-control/built-in-roles.md#cosmos-db-account-reader-role) a la identidad administrada.

```powershell
Invoke-WebRequest -Uri 'https://management.azure.com/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/<RESOURCE-GROUP>/providers/Microsoft.DocumentDb/databaseAccounts/<COSMOS DB ACCOUNT NAME>/readonlykeys/?api-version=2016-03-31' -Method POST -Headers @{Authorization="Bearer $ARMToken"}
```
En la respuesta se proporciona la lista de claves.  Por ejemplo, si obtiene las claves de solo lectura:

```powershell
{"primaryReadonlyMasterKey":"bWpDxS...dzQ==",
"secondaryReadonlyMasterKey":"38v5ns...7bA=="}
```
Ahora que tiene la clave de acceso de la cuenta de Cosmos DB, puede pasársela al SDK de Cosmos DB y realizar llamadas para acceder a la cuenta.  Como ejemplo rápido, puede pasar la clave de acceso a la CLI de Azure.  Puede obtener el valor de `<COSMOS DB CONNECTION URL>` en la pestaña **Información general** de la hoja de la cuenta de Cosmos DB en Azure Portal.  Reemplace `<ACCESS KEY>` por el valor obtenido anteriormente:

```azurecli
az cosmosdb collection show -c <COLLECTION ID> -d <DATABASE ID> --url-connection "<COSMOS DB CONNECTION URL>" --key <ACCESS KEY>
```

Este comando de la CLI devuelve detalles acerca de la colección:

```output
{
  "collection": {
    "_conflicts": "conflicts/",
    "_docs": "docs/",
    "_etag": "\"00006700-0000-0000-0000-5a8271e90000\"",
    "_rid": "Es5SAM2FDwA=",
    "_self": "dbs/Es5SAA==/colls/Es5SAM2FDwA=/",
    "_sprocs": "sprocs/",
    "_triggers": "triggers/",
    "_ts": 1518498281,
    "_udfs": "udfs/",
    "id": "Test",
    "indexingPolicy": {
      "automatic": true,
      "excludedPaths": [],
      "includedPaths": [
        {
          "indexes": [
            {
              "dataType": "Number",
              "kind": "Range",
              "precision": -1
            },
            {
              "dataType": "String",
              "kind": "Range",
              "precision": -1
            },
            {
              "dataType": "Point",
              "kind": "Spatial"
            }
          ],
          "path": "/*"
        }
      ],
      "indexingMode": "consistent"
    }
  },
  "offer": {
    "_etag": "\"00006800-0000-0000-0000-5a8271ea0000\"",
    "_rid": "f4V+",
    "_self": "offers/f4V+/",
    "_ts": 1518498282,
    "content": {
      "offerIsRUPerMinuteThroughputEnabled": false,
      "offerThroughput": 400
    },
    "id": "f4V+",
    "offerResourceId": "Es5SAM2FDwA=",
    "offerType": "Invalid",
    "offerVersion": "V2",
    "resource": "dbs/Es5SAA==/colls/Es5SAM2FDwA=/"
  }
}
```


## <a name="disable"></a>Disable

[!INCLUDE [msi-tut-disable](../../../includes/active-directory-msi-tut-disable.md)]



## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a utilizar una identidad asignada por el sistema de una máquina virtual Windows para acceder a Cosmos DB.  Para obtener más información sobre Cosmos DB, vea:

> [!div class="nextstepaction"]
>[Introducción a Azure Cosmos DB](../../cosmos-db/introduction.md)
