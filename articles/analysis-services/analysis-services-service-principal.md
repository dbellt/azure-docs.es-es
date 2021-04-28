---
title: Automatización de tareas de Azure Analysis Services con entidades de servicio | Microsoft Docs
description: Aprenda a crear una entidad de servicio para automatizar las tareas administrativas de Azure Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: owend
ms.reviewer: minewiskan
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 20902ca35cd734fd4485f59d560c605b507558d3
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108130168"
---
# <a name="automation-with-service-principals"></a>Automatización con entidades de servicio

Las entidades de servicio son un recurso de aplicación de Azure Active Directory que se crean en el inquilino para realizar operaciones desatendidas de recursos y a nivel de servicio. Son un tipo único de *identidad de usuario* con un identificador de la aplicación y una contraseña o un certificado. Una entidad de servicio tiene los permisos necesarios para realizar tareas definidas por los roles y permisos para los que se asigna. 

En Analysis Services, las entidades de servicio se usan con Azure Automation, el modo desatendido de PowerShell, las aplicaciones cliente personalizadas y las aplicaciones web para automatizar tareas comunes. Por ejemplo, las tareas de aprovisionamiento de servidores, implementación de modelos, actualización de datos, escalado vertical y reducción vertical y pausa y reanudación pueden automatizarse con el uso de las entidades de servicio. Los permisos se asignan a las entidades de servicio con la pertenencia a roles, como las cuentas de UPN de Azure AD regulares.

Analysis Services también admite las operaciones realizadas por las identidades administradas mediante entidades de servicio. Para obtener más información, vea [¿Qué es Managed Identities for Azure Resources?](../active-directory/managed-identities-azure-resources/overview.md) y [Servicios de Azure que admiten la autenticación de Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-analysis-services).    

## <a name="create-service-principals"></a>Creación de entidades de servicio
 
Las entidades de servicio se pueden crear en Azure Portal o con PowerShell. Para obtener más información, consulte:

[Creación de una entidad de servicio: Azure Portal](../active-directory/develop/howto-create-service-principal-portal.md)   
[Creación de una entidad de servicio: PowerShell](../active-directory/develop/howto-authenticate-service-principal-powershell.md)

## <a name="store-credential-and-certificate-assets-in-azure-automation"></a>Almacenamiento de recursos de credenciales y certificados en Azure Automation

Las credenciales y los certificados de las entidades de servicio se pueden almacenar de forma segura en Azure Automation para las operaciones de runbook. Para obtener más información, consulte:

[Recursos de credenciales en Azure Automation](../automation/shared-resources/credentials.md)   
[Recursos de certificados en Azure Automation](../automation/shared-resources/certificates.md)

## <a name="add-service-principals-to-server-admin-role"></a>Incorporación de entidades de servicio al rol de administrador del servidor

Antes de poder usar una entidad de servicio para las operaciones de administración del servidor de Analysis Services, debe agregarla al rol de administradores del servidor. Las entidades de servicio deben agregarse directamente al rol Administrador del servidor. La incorporación de una entidad de servicio a un grupo de seguridad y, posteriormente, de ese grupo de seguridad al rol Administrador del servidor, no se admite. Para más información, vea [Incorporación de una entidad de servicio al rol de administrador del servidor](analysis-services-addservprinc-admins.md).

## <a name="service-principals-in-connection-strings"></a>Entidades de servicio en las cadenas de conexión

El identificador de aplicación y la contraseña o el certificado de las entidades de servicio se pueden usar en las cadenas de conexión igual que un UPN.

### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

#### <a name="using-azanalysisservices-module"></a><a name="azmodule"></a>Uso del módulo Az.AnalysisServices

Al usar una entidad de servicio para las operaciones de administración de recursos con el módulo [Az.AnalysisServices](/powershell/module/az.analysisservices), emplee el cmdlet `Connect-AzAccount`. 

En el ejemplo siguiente, se usan appID y una contraseña para realizar operaciones de plano de control para la sincronización en réplicas de solo lectura y el escalado vertical u horizontal:

```powershell
Param (
        [Parameter(Mandatory=$true)] [String] $AppId,
        [Parameter(Mandatory=$true)] [String] $PlainPWord,
        [Parameter(Mandatory=$true)] [String] $TenantId
       )
$PWord = ConvertTo-SecureString -String $PlainPWord -AsPlainText -Force
$Credential = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList $AppId, $PWord

# Connect using Az module
Connect-AzAccount -Credential $Credential -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxx"

# Syncronize a database for query scale out
Sync-AzAnalysisServicesInstance -Instance "asazure://westus.asazure.windows.net/testsvr" -Database "testdb"

# Scale up the server to an S1, set 2 read-only replicas, and remove the primary from the query pool. The new replicas will hydrate from the synchronized data.
Set-AzAnalysisServicesServer -Name "testsvr" -ResourceGroupName "testRG" -Sku "S1" -ReadonlyReplicaCount 2 -DefaultConnectionMode Readonly
```

#### <a name="using-sqlserver-module"></a>Uso del módulo SQLServer

En el ejemplo siguiente, se usa el identificador de aplicación y una contraseña para realizar una operación de actualización de la base de datos modelo.

```powershell
Param (
        [Parameter(Mandatory=$true)] [String] $AppId,
        [Parameter(Mandatory=$true)] [String] $PlainPWord,
        [Parameter(Mandatory=$true)] [String] $TenantId
       )
$PWord = ConvertTo-SecureString -String $PlainPWord -AsPlainText -Force

$Credential = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList $AppId, $PWord

Invoke-ProcessTable -Server "asazure://westcentralus.asazure.windows.net/myserver" -TableName "MyTable" -Database "MyDb" -RefreshType "Full" -ServicePrincipal -ApplicationId $AppId -TenantId $TenantId -Credential $Credential
```

### <a name="amo-and-adomd"></a>AMO y ADOMD 

Al conectarse con aplicaciones cliente y aplicaciones web, los paquetes instalables de la versión 15.0.2 y versiones posteriores de las [bibliotecas cliente de AMO y ADOMD](/analysis-services/client-libraries?view=azure-analysis-services-current&preserve-view=true) de NuGet admiten entidades de servicio en las cadenas de conexión con el uso de la siguiente sintaxis: `app:AppID` y la contraseña o `cert:thumbprint`. 

En el ejemplo siguiente, se usan un `appID` y una `password` para realizar una operación de actualización de la base de datos modelo:

```csharp
string appId = "xxx";
string authKey = "yyy";
string connString = $"Provider=MSOLAP;Data Source=asazure://westus.asazure.windows.net/<servername>;User ID=app:{appId};Password={authKey};";
Server server = new Server();
server.Connect(connString);
Database db = server.Databases.FindByName("adventureworks");
Table tbl = db.Model.Tables.Find("DimDate");
tbl.RequestRefresh(RefreshType.Full);
db.Model.SaveChanges();
```

## <a name="next-steps"></a>Pasos siguientes
[Inicio de sesión con Azure PowerShell](/powershell/azure/authenticate-azureps)   
[Actualización con Logic Apps](analysis-services-refresh-logic-app.md)  
[Actualización con Azure Automation](analysis-services-refresh-azure-automation.md)  
[Incorporación de una entidad de servicio al rol de administrador del servidor](analysis-services-addservprinc-admins.md)  
[Automatización de las tareas del conjunto de datos y el área de trabajo de Power BI Premium con entidades de servicio](/power-bi/admin/service-premium-service-principal)
