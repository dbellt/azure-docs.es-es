---
title: Habilitación de una identidad administrada para la cuenta de Azure Automation (versión preliminar)
description: En este artículo se describe cómo configurar la identidad administrada para las cuentas de Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 04/14/2021
ms.topic: conceptual
ms.openlocfilehash: 93c55c21bf740f2851cac1926bc673cebcd914b0
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107514808"
---
# <a name="enable-a-managed-identity-for-your-azure-automation-account-preview"></a>Habilitación de una identidad administrada para la cuenta de Azure Automation (versión preliminar)

En este tema se muestra cómo crear una identidad administrada para una cuenta de Azure Automation y usarla para acceder a otros recursos. Para más información sobre cómo funciona la identidad administrada con Azure Automation, consulte [Identidades administradas](automation-security-overview.md#managed-identities-preview).

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta y una suscripción de Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar. Tanto la identidad administrada como los recursos de Azure de destino que administra el runbook con ella deben estar en la misma suscripción de Azure.

- La versión más reciente de los módulos de cuenta de Azure Automation. Actualmente, es la versión 1.6.0. (Consulte [Az.Automation 1.6.0](https://www.powershellgallery.com/packages/Az.Automation/1.6.0) para más información sobre esta versión).

- Un recurso de Azure al que desea acceder desde el runbook de Automation. Este recurso debe tener un rol definido para la identidad administrada, lo que ayuda al runbook de Automation a autenticar el acceso al recurso. Para agregar roles, debe ser propietario del recurso en el inquilino de Azure AD correspondiente.

- Si desea ejecutar trabajos híbridos mediante una identidad administrada, actualice Hybrid Runbook Worker a la versión más reciente. Las versiones mínimas obligatorias son:

   - Hybrid Runbook Worker de Windows: versión 7.3.1125.0
   - Hybrid Runbook Worker de Linux: versión 1.7.4.0

## <a name="enable-system-assigned-identity"></a>Activar una identidad asignada por el sistema

>[!IMPORTANT]
>La nueva identidad de nivel de cuenta de Automation invalidará las identidades asignadas por el sistema de nivel de máquina virtual anteriores (que se describen en [Uso de la autenticación de runbook con identidades administradas](/automation-hrw-run-runbooks#runbook-auth-managed-identities)). Si ejecuta trabajos híbridos en máquinas virtuales de Azure que usan la identidad asignada por el sistema de la máquina virtual para acceder a los recursos del runbook, la identidad de la cuenta de Automation se usará para los trabajos híbridos. Esto significa que la ejecución del trabajo existente puede verse afectada si ha estado usando la característica Claves administradas de cliente (CMK) de la cuenta de Automation.<br/><br/>Si desea seguir usando la identidad administrada de la máquina virtual, no debe habilitar la identidad de nivel de cuenta de Automation. Si ya la ha habilitado, puede deshabilitar la identidad administrada de la cuenta de Automation. Consulte [Deshabilitación de la identidad administrada de la cuenta de Azure Automation.](https://docs.microsoft.com/azure/automation/disable-managed-identity-for-automation)

La configuración de identidades asignadas por el sistema para Azure Automation se puede realizar de dos maneras. Puede usar Azure Portal o la API REST de Azure.

>[!NOTE]
>Las identidades asignadas por el usuario aún no se admiten.

### <a name="enable-system-assigned-identity-in-azure-portal"></a>Activar una identidad asignada por el sistema en Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Vaya a la cuenta de Automation y seleccione **Identidad** en **Configuración de cuenta**.

1. Establezca la opción **Asignado por el sistema** en **Activado** y presione **Guardar**. Cuando se le solicite confirmación, seleccione **Sí**.

:::image type="content" source="media/managed-identity/managed-identity-on.png" alt-text="Habilitación de una identidad asignada por el sistema en Azure Portal.":::

Ahora la cuenta de Automation podrá usar la identidad asignada por el sistema, registrada con Azure Active Directory (Azure AD) y representada mediante un identificador de objeto.

:::image type="content" source="media/managed-identity/managed-identity-object-id.png" alt-text="Identificador de objeto de la identidad administrada.":::

### <a name="enable-system-assigned-identity-through-the-rest-api"></a>Habilitación de la identidad asignada por el sistema mediante la API REST

Puede configurar una identidad administrada asignada por el sistema a la cuenta de Automation mediante la siguiente llamada a la API REST.

```http
PATCH https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resource-group-name/providers/Microsoft.Automation/automationAccounts/automation-account-name?api-version=2020-01-13-preview
```

Cuerpo de la solicitud
```json
{ 
 "identity": 
 { 
  "type": "SystemAssigned" 
  } 
}
```

```json
{
 "name": "automation-account-name",
 "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resource-group-name/providers/Microsoft.Automation/automationAccounts/automation-account-name",
 .
 .
 "identity": {
    "type": "SystemAssigned",
    "principalId": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
    "tenantId": "bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb"
 },
.
.
}
```

|Propiedad (JSON) | Value | Descripción|
|----------|-----------|------------|
| principalid | \<principal-ID\> | Identificador único global (GUID) del objeto de entidad de servicio para la identidad administrada que representa la cuenta de Automation en el inquilino de Azure AD. Este GUID aparece a veces como "Id. de objeto" u objectID. |
| tenantid | \<Azure-AD-tenant-ID\> | Identificador único global (GUID) que representa el inquilino de Azure AD del que es ahora miembro la cuenta de Automation. En el inquilino de Azure AD, la entidad de servicio tiene el mismo nombre que la cuenta de Automation. |

## <a name="give-identity-access-to-azure-resources-by-obtaining-a-token"></a>Concesión de acceso a los recursos de Azure mediante un token

Una cuenta de Automation puede utilizar su identidad administrada para obtener tokens de acceso a otros recursos protegidos por Azure AD, como Azure Key Vault. Estos tokens no representan a ningún usuario específico de la aplicación, sino que representan la aplicación que accede al recurso. En este caso, por ejemplo, el token representa una cuenta de Automation.

Para poder usar la identidad administrada asignada por el sistema para la autenticación, configure el acceso para esa identidad en el recurso de Azure en el que planea usar la identidad. Para completar esta tarea, asigne el rol adecuado a esa identidad en el recurso de Azure de destino.

En este ejemplo se usa Azure PowerShell para mostrar cómo asignar el rol Colaborador en la suscripción al recurso de Azure de destino. El rol Colaborador se usa como ejemplo y puede ser necesario o no en su caso.

```powershell
New-AzRoleAssignment -ObjectId <automation-Identity-object-id> -Scope "/subscriptions/<subscription-id>" -RoleDefinitionName "Contributor"
```

## <a name="authenticate-access-with-managed-identity"></a>Autenticación del acceso con una identidad administrada

Después de habilitar la identidad administrada para la cuenta de Automation y de conceder acceso a una identidad al recurso de destino, puede especificar esa identidad en los runbooks para los recursos que admiten la identidad administrada. Para compatibilidad con las identidades, use el cmdlet `Connect-AzAccount`. Consulte [Connect-AzAccount](/powershell/module/az.accounts/Connect-AzAccount) en la referencia de PowerShell.

```powershell
Connect-AzAccount -Identity
```

>[!NOTE]
>Si su organización sigue usando los cmdlets de AzureRM en desuso, puede usar `Connect-AzureRMAccount -Identity`.

## <a name="generate-an-access-token-without-using-azure-cmdlets"></a>Generación de un token de acceso sin usar cmdlets de Azure

Para los puntos de conexión HTTP, asegúrese de lo siguiente.
- El encabezado de metadatos debe estar presente y establecido en "true".
- Se debe pasar un recurso junto con la solicitud, como parámetro de consulta para una solicitud GET y como datos de formulario para una solicitud POST.
- X-IDENTITY-HEADER debe establecerse en el valor de la variable de entorno IDENTITY_HEADER para Hybrid Runbook Worker. 
- El tipo de contenido de la solicitud Post debe ser "application/x-www-form-urlencoded". 

### <a name="sample-get-request"></a>Solicitud GET de ejemplo

```powershell
$resource= "?resource=https://management.azure.com/" 
$url = $env:IDENTITY_ENDPOINT + $resource 
$Headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]" 
$Headers.Add("X-IDENTITY-HEADER", $env:IDENTITY_HEADER) 
$Headers.Add("Metadata", "True") 
$accessToken = Invoke-RestMethod -Uri $url -Method 'GET' -Headers $Headers
Write-Output $accessToken.access_token
```

### <a name="sample-post-request"></a>Solicitud POST de ejemplo
```powershell
$url = $env:IDENTITY_ENDPOINT  
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]" 
$headers.Add("X-IDENTITY-HEADER", $env:IDENTITY_HEADER) 
$headers.Add("Metadata", "True") 
$body = @{resource='https://management.azure.com/' } 
$accessToken = Invoke-RestMethod $url -Method 'POST' -Headers $headers -ContentType 'application/x-www-form-urlencoded' -Body $body 
Write-Output $accessToken.access_token
```

## <a name="sample-runbooks-using-managed-identity"></a>Runbooks de ejemplo con identidad administrada

### <a name="sample-runbook-to-access-a-sql-database-without-using-azure-cmdlets"></a>Runbook de ejemplo para acceder a una base de datos SQL sin usar cmdlets de Azure

```powershell
$queryParameter = "?resource=https://database.windows.net/" 
$url = $env:IDENTITY_ENDPOINT + $queryParameter
$Headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]" 
$Headers.Add("X-IDENTITY-HEADER", $env:IDENTITY_HEADER) 
$Headers.Add("Metadata", "True") 
$content =[System.Text.Encoding]::Default.GetString((Invoke-WebRequest -UseBasicParsing -Uri $url -Method 'GET' -Headers $Headers).RawContentStream.ToArray()) | ConvertFrom-Json 
$Token = $content.access_token 
echo "The managed identities for Azure resources access token is $Token" 
$SQLServerName = "<ServerName>"    # Azure SQL logical server name  
$DatabaseName = "<DBname>"     # Azure SQL database name 
Write-Host "Create SQL connection string" 
$conn = New-Object System.Data.SqlClient.SQLConnection  
$conn.ConnectionString = "Data Source=$SQLServerName.database.windows.net;Initial Catalog=$DatabaseName;Connect Timeout=30" 
$conn.AccessToken = $Token 
Write-host "Connect to database and execute SQL script" 
$conn.Open()  
$ddlstmt = "CREATE TABLE Person( PersonId INT IDENTITY PRIMARY KEY, FirstName NVARCHAR(128) NOT NULL)" 
Write-host " " 
Write-host "SQL DDL command" 
$ddlstmt 
$command = New-Object -TypeName System.Data.SqlClient.SqlCommand($ddlstmt, $conn) 
Write-host "results" 
$command.ExecuteNonQuery() 
$conn.Close()
```

### <a name="sample-runbook-to-access-a-key-vault-using-azure-cmdlets"></a>Runbook de ejemplo para acceder a un almacén de claves mediante cmdlets de Azure

```powershell
Write-Output "Connecting to azure via  Connect-AzAccount -Identity" 
Connect-AzAccount -Identity 
Write-Output "Successfully connected with Automation account's Managed Identity" 
Write-Output "Trying to fetch value from key vault using MI. Make sure you have given correct access to Managed Identity" 
$secret = Get-AzKeyVaultSecret -VaultName '<KVname>' -Name '<KeyName>' 

$ssPtr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($secret.SecretValue) 
try { 
  $secretValueText = [System.Runtime.InteropServices.Marshal]::PtrToStringBSTR($ssPtr) 
    Write-Output $secretValueText 
} finally { 
    [System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($ssPtr) 
}
```

### <a name="sample-python-runbook-to-get-a-token"></a>Runbook de Python de ejemplo para obtener un token
 
```python
#!/usr/bin/env python3 
import os 
import requests  
# printing environment variables 
endPoint = os.getenv('IDENTITY_ENDPOINT')+"?resource=https://management.azure.com/" 
identityHeader = os.getenv('IDENTITY_HEADER') 
payload={} 
headers = { 
  'X-IDENTITY-HEADER': identityHeader,
  'Metadata': 'True' 
} 
response = requests.request("GET", endPoint, headers=headers, data=payload) 
print(response.text) 
```

## <a name="next-steps"></a>Pasos siguientes

- Si necesita deshabilitar una identidad administrada, consulte [Deshabilitación de la identidad administrada de la identidad administrada de la cuenta de Azure Automation (versión preliminar)](disable-managed-identity-for-automation.md).

- Para información general sobre la seguridad de l cuenta de Azure Automation, consulte [Introducción a la autenticación de cuentas de Automation](automation-security-overview.md).