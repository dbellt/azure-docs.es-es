---
title: Obtención de los valores necesarios para autenticar una aplicación
description: Cree una entidad de servicio para acceder a Azure SQL Database desde el código.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=1 , devx-track-azurecli, devx-track-azurepowershell
ms.devlang: ''
ms.topic: how-to
author: VanMSFT
ms.author: vanto
ms.reviewer: mathoma
ms.date: 03/12/2019
ms.openlocfilehash: 923e218768e10fcf31c5dd1f3408c6dfc1af27b2
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110705694"
---
# <a name="get-the-required-values-for-authenticating-an-application-to-access-azure-sql-database-from-code"></a>Obtención de los valores necesarios para autenticar una aplicación y acceder a Azure SQL Database desde el código
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Para crear y administrar Azure SQL Database desde el código, debe registrar la aplicación en el dominio de Azure Active Directory (Azure AD) en la suscripción donde se han creado los recursos de Azure.

## <a name="create-a-service-principal-to-access-resources-from-an-application"></a>Creación de una entidad de servicio para acceder a recursos desde una aplicación

En los ejemplos siguientes se crea la aplicación de Active Directory (AD) y la entidad de servicio que se necesitan para autenticar la aplicación de C#. En la salida del script, se encuentran los valores que se necesitan para el anterior ejemplo de C#. Para ver información detallada, consulte [Uso de Azure PowerShell para crear una entidad de servicio para acceder a recursos](../../active-directory/develop/howto-authenticate-service-principal-powershell.md).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

> [!IMPORTANT]
> El módulo de Azure Resource Manager (RM) para PowerShell todavía es compatible con SQL Database, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. El módulo de AzureRM continuará recibiendo correcciones de errores hasta diciembre de 2020 como mínimo.  Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos. Para obtener más información sobre la compatibilidad, vea [Presentación del nuevo módulo Az de Azure PowerShell](/powershell/azure/new-azureps-module-az).

```powershell
# sign in to Azure
Connect-AzAccount

# for multiple subscriptions, uncomment and set to the subscription you want to work with
#$subscriptionId = "{xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}"
#Set-AzContext -SubscriptionId $subscriptionId

$appName = "{app-name}" # display name for your app, must be unique in your directory
$uri = "http://{app-name}" # does not need to be a real uri
$secret = "{app-password}"

# create an AAD app
$azureAdApplication = New-AzADApplication -DisplayName $appName -HomePage $Uri -IdentifierUris $Uri -Password $secret

# create a Service Principal for the app
$svcprincipal = New-AzADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId

Start-Sleep -s 15 # to avoid a PrincipalNotFound error, pause here for 15 seconds

# if you still get a PrincipalNotFound error, then rerun the following until successful.
$roleassignment = New-AzRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $azureAdApplication.ApplicationId.Guid

# output the values we need for our C# application to successfully authenticate
Write-Output "Copy these values into the C# sample app"

Write-Output "_subscriptionId:" (Get-AzContext).Subscription.SubscriptionId
Write-Output "_tenantId:" (Get-AzContext).Tenant.TenantId
Write-Output "_applicationId:" $azureAdApplication.ApplicationId.Guid
Write-Output "_applicationSecret:" $secret
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
# sign in to Azure
az login

# for multiple subscriptions, uncomment and set to the subscription you want to work with
#$subscriptionId = "{xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}"
#az account set --subscription $subscriptionId

$appName = "{app-name}" # display name for your app, must be unique in your directory
$uri = "http://{app-name}" # does not need to be a real uri
$secret = "{app-password}"

# create an AAD app
$azureAdApplication = az ad app create --display-name $appName --homepage $Uri --identifier-uris $Uri --password $secret

# create a Service Principal for the app
$svcprincipal = az ad sp create --id $azureAdApplication.ApplicationId

Start-Sleep -s 15 # to avoid a PrincipalNotFound error, pause for 15 seconds

# if you still get a PrincipalNotFound error, then rerun the following until successful.
$roleassignment = az role assignment create --role "Contributor" --assignee $azureAdApplication.ApplicationId.Guid

# output the values we need for our C# application to successfully authenticate
Write-Output "Copy these values into the C# sample app"

Write-Output "_subscriptionId:" (az account show --query "id")
Write-Output "_tenantId:" (az account show --query "tenantId")
Write-Output "_applicationId:" $azureAdApplication.ApplicationId.Guid
Write-Output "_applicationSecret:" $secret
```

* * *

## <a name="see-also"></a>Consulte también

[Creación de una base de datos en Azure SQL Database con C#](design-first-database-csharp-tutorial.md)  
[Conexión a Azure SQL Database mediante la autenticación de Azure Active Directory](authentication-aad-overview.md)
