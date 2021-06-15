---
title: Clasificación de los datos de Azure SQL mediante etiquetas de Azure Purview
description: Importación de la clasificación de Azure Purview en Azure SQL Database y Azure Synpase Analytics
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: azurepowershell
ms.topic: sample
author: davidtrigano
ms.author: datrigan
ms.reviewer: vanto, mathoma
ms.date: 02/17/2021
ms.openlocfilehash: d1064c6e60c8bdc80251228ddc2a36b5c3a77852
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110694811"
---
# <a name="classify-your-azure-sql-data-using-azure-purview-labels"></a>Clasificación de los datos de Azure SQL mediante etiquetas de Azure Purview
[!INCLUDE[appliesto-sqldb-asa](../../azure-sql/includes/appliesto-sqldb-asa.md)]

En este documento se describe cómo agregar etiquetas de Azure Purview a Azure SQL Database y Azure Synapse Analytics (anteriormente SQL DW).

## <a name="create-an-application"></a>Crear una aplicación

1. En Azure Portal, abra **Azure Active Directory**.
2. En **Administrar**, seleccione **Registro de aplicación**.
3. Cree una aplicación de Azure Active Directory; para ello, seleccione **Nueva aplicación**.
4. Escriba un nombre para la aplicación y seleccione **Registrar**.
5. Una vez creada la aplicación, Abra **Certificates & secrets** (Certificados y secretos) en **Administrador**.
6. Para crear un secreto de cliente, seleccione **Nuevo secreto de cliente** en **Secretos de cliente**.
7. Agregue una descripción al secreto de cliente, seleccione un período de expiración y seleccione **Agregar**.
8. Mantenga el **valor** para usarlo en el futuro.

   > [!NOTE]
   > Una vez cerrada la página, el valor se ocultará. No podrá recuperar el valor del secreto del cliente si vuelve a la página. Tendrá que generar un nuevo secreto de cliente.

9. Vuelva a la página de información general de la aplicación recién creada y copie los valores siguientes para usarlos en el futuro:
    1. Id. de aplicación (cliente)
    1. Id. de directorio (inquilino)

## <a name="provide-permissions-to-the-application"></a>Asignación de permisos a la aplicación

1. En Azure Portal, busque **Purview accounts** (Cuentas de Purview).
2. Seleccione la cuenta de Purview en la que se clasifican las bases de datos SQL y Synapse.
3. Abra **Control de acceso (IAM)** y seleccione **Agregar**.

4. Seleccione **Agregar asignación de roles**.
5. En la sección **Rol**, busque **Purview Data Reader** (Lector de datos de Purview) y selecciónelo.
6. En la sección **Seleccionar**, busque la aplicación que creó anteriormente, selecciónela y presione **Guardar**.

## <a name="extract-the-classification-from-azure-purview"></a>Extracción de la clasificación de Azure Purview

1. Abra su cuenta de Purview y, en la página principal, busque la instancia de Azure SQL Database o Azure Synapse Analytics donde desee copiar las etiquetas.
2. Copie el qualifiedName en **Propiedades** y guárdelo para usarlo en el futuro.
3. Abra el shell de PowerShell.

4. Copie uno de los siguientes scripts según el tipo de recurso de SQL (Azure SQL Database o Azure Synapse).
5. Rellene los parámetros con los valores que copió anteriormente:

   a. $TenantID: sección 1, paso 9
   
   b. $ClientID: sección 1, paso 9
   
   c. $SecretID: sección 1, paso 8
   
   d. $purviewAccountName: sección 2, paso 2
   
   e. $sqlDatabaseName: sección 3, paso 2

6. Copie la salida del script para usarla en el futuro.

### <a name="for-azure-sql-database"></a>En Azure SQL Database

```powershell
# Replace the values below with the relevant values for your environment
$TenantID = 'your_tenant_id'
$ClientID = 'your_client_id'
$SecretID = 'your_secret_id'
$purviewAccountName='purview_account_name'
$sqlDatabaseName="mssql://sql_server_name.database.windows.net/db_name"

###############################################################################
# Get an access accessToken, and build REST Request Header.
###############################################################################
$cmdletParameters = @{
  Method  = "POST"
  URI     = "https://login.microsoftonline.com/$TenantID/oauth2/token"
  Headers = @{
    "Content-Type" = 'application/x-www-form-urlencoded'
  }
  Body    = "client_id=$ClientID&client_secret=$SecretID&resource=https://purview.azure.net&grant_type=client_credentials"
}

$invokeResult = Invoke-RestMethod @cmdletParameters;
$accessToken = $invokeResult.access_token; 

$restRequestHeader = @{
  "authorization" = "Bearer $accessToken"
};

###############################################################################
# Get database entity.
###############################################################################
$getDatabaseEntityEntryEndpoint = "https://" + $purviewAccountName + ".catalog.purview.azure.com/api/atlas/v2/entity/uniqueAttribute/type/azure_sql_db?attr:qualifiedName=" + $sqlDatabaseName;
$cmdletParameters = @{
  Method      = "Get"
  Uri         = $getDatabaseEntityEntryEndpoint
  Headers     = $restRequestHeader
  ContentType = "application/json"        
};

$invokeResult =  Invoke-RestMethod @cmdletParameters;
$referredEntities = $invokeResult.referredEntities;
if ($null -eq $referredEntities) {
  Write-Output "No referred entities found under database entity!";
  exit;
}

###############################################################################
# Iterate database referred entities, find classified columns, and build T-SQL.
###############################################################################
foreach ($referredEntity in $referredEntities.psobject.Properties.GetEnumerator()) {
  $typeName = $referredEntity.Value.typeName;
  if ($null -eq $typeName -or $typeName -ne 'azure_sql_column') {
    continue;
  }
  
  $classifications = $referredEntity.Value.classifications;
  if ($null -eq $classifications) {
    continue;
  }
  
  foreach ($classification in $classifications.GetEnumerator())
  {
    if ($classification.typeName -notmatch 'Microsoft\.Label\.(?<labelId>.+)') { 
        continue;
    }

      $labelId = $Matches.labelId -replace "_", "-";

      $attributes = $referredEntity.Value.attributes;
      if ($null -eq $attributes) {
        continue;
      }
    
      $qualifiedName = $attributes.qualifiedName;
      if ($qualifiedName -notmatch '.+\.database\.windows\.net\/.+\/(?<schemaName>.+)\/(?<tableName>.+)#(?<columnName>.+)') {
        continue;
      }

      $schemaName = $Matches.schemaName;
      $tableName = $Matches.tableName;
      $columnName = $Matches.columnName;
      
      Write-Output "ADD SENSITIVITY CLASSIFICATION TO ${schemaName}.${tableName}.${columnName} WITH (LABEL='Azure Purview Label', LABEL_ID='${labelId}');";
  }
}
```

### <a name="for-azure-synapse-analytics"></a>En Azure Synapse Analytics

```powershell
# Replace the values below with the relevant values for your environment
$TenantID = 'your_tenant_id'
$ClientID = 'your_client_id'
$SecretID = 'your_secret_id'
$purviewAccountName='purview_account_name'
$dwDatabaseName="mssql://dw_server_name.database.windows.net/dw_name"

###############################################################################
# Get an access accessToken, and build REST Request Header.
###############################################################################
$cmdletParameters = @{
  Method  = "POST"
  URI     = "https://login.microsoftonline.com/$TenantID/oauth2/token"
  Headers = @{
    "Content-Type" = 'application/x-www-form-urlencoded'
  }
  Body    = "client_id=$ClientID&client_secret=$SecretID&resource=https://purview.azure.net&grant_type=client_credentials"
}

$invokeResult = Invoke-RestMethod @cmdletParameters;
$accessToken = $invokeResult.access_token; 

$restRequestHeader = @{
  "authorization" = "Bearer $accessToken"
};

###############################################################################
# Get database entity.
###############################################################################
$getDatabaseEntityEntryEndpoint = "https://" + $purviewAccountName + ".catalog.purview.azure.com/api/atlas/v2/entity/uniqueAttribute/type/azure_sql_dw?attr:qualifiedName=" + $dwDatabaseName;
$cmdletParameters = @{
  Method      = "Get"
  Uri         = $getDatabaseEntityEntryEndpoint
  Headers     = $restRequestHeader
  ContentType = "application/json"        
};

$invokeResult =  Invoke-RestMethod @cmdletParameters;
$referredEntities = $invokeResult.referredEntities;
if ($null -eq $referredEntities) {
  Write-Output "No referred entities found under database entity!";
  exit;
}

###############################################################################
# Iterate database referred entities, find classified columns, and build T-SQL.
###############################################################################
foreach ($referredEntity in $referredEntities.psobject.Properties.GetEnumerator()) {
  $typeName = $referredEntity.Value.typeName;
  if ($null -eq $typeName -or $typeName -ne 'azure_sql_dw_column') {
    continue;
  }
  
  $classifications = $referredEntity.Value.classifications;
  if ($null -eq $classifications) {
    continue;
  }
  
  foreach ($classification in $classifications.GetEnumerator())
  {
    if ($classification.typeName -notmatch 'Microsoft\.Label\.(?<labelId>.+)') { 
        continue;
    }

      $labelId = $Matches.labelId -replace "_", "-";

      $attributes = $referredEntity.Value.attributes;
      if ($null -eq $attributes) {
        continue;
      }
    
      $qualifiedName = $attributes.qualifiedName;
      if ($qualifiedName -notmatch '.+\.database\.windows\.net\/.+\/(?<schemaName>.+)\/(?<tableName>.+)#(?<columnName>.+)') {
        continue;
      }

      $schemaName = $Matches.schemaName;
      $tableName = $Matches.tableName;
      $columnName = $Matches.columnName;
      
      Write-Output "ADD SENSITIVITY CLASSIFICATION TO ${schemaName}.${tableName}.${columnName} WITH (LABEL='Azure Purview Label', LABEL_ID='${labelId}');";
  }
}
```

## <a name="run-the-t-sql-command-on-your-sql-asset"></a>Ejecute el comando T-SQL en el recurso de SQL.

1. Conéctese a su instancia de Azure SQL Database o Azure Synapse con la herramienta que prefiera.
2. Ejecute el comando T-SQL que copió de la sección anterior.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/).

Para más información sobre Azure Purview, consulte la [Documentación de Azure Purview](../../purview/index.yml).