---
title: Solución de problemas de identidades administradas de Azure Automation (versión preliminar)
description: En este artículo se explica cómo solucionar problemas al usar una identidad administrada con una cuenta de Automation.
services: automation
ms.subservice: ''
ms.date: 04/28/2021
ms.author: v-dturcaso
ms.topic: troubleshooting
ms.openlocfilehash: f42605b92093cf08a417690eae8324b9280e8ce6
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108230092"
---
# <a name="troubleshoot-azure-automation-managed-identity-issues-preview"></a>Solución de problemas de identidades administradas de Azure Automation (versión preliminar)

En este artículo se describen soluciones para los problemas que pueden surgir al usar una identidad administrada con la cuenta de Automation. Para información general sobre el uso de identidades administradas con cuentas de Automation, consulte [Introducción a la autenticación de cuentas de Azure Automation](../automation-security-overview.md#managed-identities-preview).

## <a name="scenario-attempt-to-use-managed-identity-with-automation-account-fails"></a>Escenario: Se produce un error al intentar usar la identidad administrada con la cuenta de Automation

### <a name="issue"></a>Problema

Al intentar trabajar con identidades administradas en la cuenta de Automation, se produce un error como el siguiente:

```error
Connect-AzureRMAccount : An error occurred while sending the request. At line:2 char:1 + Connect-AzureRMAccount -Identity + 
CategoryInfo : CloseError: (:) [Connect-AzureRmAccount], HttpRequestException + FullyQualifiedErrorId : Microsoft.Azure.Commands.Profile.ConnectAzureRmAccountCommand
```

### <a name="cause"></a>Causa

La causa más común es que no haya habilitado la identidad antes de intentar usarla. Para comprobarlo, ejecute el siguiente runbook de PowerShell en la cuenta de Automation afectada.

```powershell
resource= "?resource=https://management.azure.com/"
$url = $env:IDENTITY_ENDPOINT + $resource
$Headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$Headers.Add("X-IDENTITY-HEADER", $env:IDENTITY_HEADER)
$Headers.Add("Metadata", "True")

try
{
    $Response = Invoke-RestMethod -Uri $url -Method 'GET' -Headers $Headers
}
catch
{
    $StatusCode = $_.Exception.Response.StatusCode.value__
    $stream = $_.Exception.Response.GetResponseStream()
    $reader = New-Object System.IO.StreamReader($stream)
    $responseBody = $reader.ReadToEnd()
    
    Write-Output "Request Failed with Status: $StatusCode, Message: $responseBody"
}
```

Si el problema era ese, debería ver un resultado similar al siguiente:

`Request Failed with Status: 400, Message: {"Message":"No managed identity was found for Automation account xxxxxxxxxxxx"}`

### <a name="resolution"></a>Solución

Antes de poder usar el servicio de identidad administrada, debe habilitar una identidad para la cuenta de Automation. Consulte [Habilitación de una identidad administrada para la cuenta de Azure Automation (versión preliminar)](../enable-managed-identity-for-automation.md).

## <a name="next-steps"></a>Pasos siguientes

Si en este artículo no se resuelve la incidencia, pruebe uno de los canales siguientes para obtener ayuda adicional:

* Obtenga respuestas de expertos de Azure en los [foros de Azure](https://azure.microsoft.com/support/forums/).
* Conectar con [@AzureSupport](https://twitter.com/azuresupport). Se trata de la cuenta oficial de Microsoft Azure para conectar a la comunidad de Azure con los recursos adecuados: respuestas, soporte técnico y expertos.
* Registrar un incidente de soporte técnico de Azure. Vaya al [sitio de Soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte técnico**.