---
title: Envío de un correo electrónico desde un runbook de Azure Automation
description: En este artículo se indica cómo enviar un correo electrónico desde un runbook.
services: automation
ms.subservice: process-automation
ms.date: 01/05/2021
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: a2650e3a9ce58b611c1aff1a569cc1e8f0980fd4
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107833497"
---
# <a name="send-an-email-from-a-runbook"></a>Envío de un correo electrónico desde un runbook

Puede enviar un correo electrónico desde un runbook con [SendGrid](https://sendgrid.com/solutions) mediante PowerShell. 

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción de Azure. Si aún no tiene ninguna, puede [activar las ventajas de la suscripción a MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o [suscribirse para obtener una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Una cuenta de SendGrid](../sendgrid-dotnet-how-to-send-email.md#create-a-sendgrid-account).
* [Una cuenta de Automation](./index.yml) con módulos **Az**.
* [Una cuenta de ejecución](./automation-security-overview.md#run-as-accounts) para almacenar y ejecutar el runbook.

## <a name="create-an-azure-key-vault"></a>Crear una instancia de Azure Key Vault

Puede crear una instancia de Azure Key Vault mediante el siguiente script de PowerShell. Reemplace los valores variables por valores específicos de su entorno. Use Azure Cloud Shell integrado a través del botón **Pruébelo**, situado en la esquina superior derecha del bloque de código. También puede copiar y ejecutar el código localmente si tiene los [módulos de Az](/powershell/azure/install-az-ps) instalados en la máquina local.

> [!NOTE]
> Para recuperar la clave de API, siga los pasos para [encontrar la clave de API de SendGrid](../sendgrid-dotnet-how-to-send-email.md#to-find-your-sendgrid-api-key).

```azurepowershell-interactive
$SubscriptionId  =  "<subscription ID>"

# Sign in to your Azure account and select your subscription
# If you omit the SubscriptionId parameter, the default subscription is selected.
Connect-AzAccount -SubscriptionId $SubscriptionId

# Use Get-AzLocation to see your available locations.
$region = "southcentralus"
$KeyVaultResourceGroupName  = "mykeyvaultgroup"
$VaultName = "<Enter a universally unique vault name>"
$SendGridAPIKey = "<SendGrid API key>"
$AutomationAccountName = "testaa"

# Create new Resource Group, or omit this step if you already have a resource group.
New-AzResourceGroup -Name $KeyVaultResourceGroupName -Location $region

# Create the new key vault
$newKeyVault = New-AzKeyVault -VaultName $VaultName -ResourceGroupName $KeyVaultResourceGroupName -Location $region
$resourceId = $newKeyVault.ResourceId

# Convert the SendGrid API key into a SecureString
$Secret = ConvertTo-SecureString -String $SendGridAPIKey -AsPlainText -Force
Set-AzKeyVaultSecret -VaultName $VaultName -Name 'SendGridAPIKey' -SecretValue $Secret

# Grant access to the Key Vault to the Automation Run As account.
$connection = Get-AzAutomationConnection -ResourceGroupName $KeyVaultResourceGroupName -AutomationAccountName $AutomationAccountName -Name AzureRunAsConnection
$appID = $connection.FieldDefinitionValues.ApplicationId
Set-AzKeyVaultAccessPolicy -VaultName $VaultName -ServicePrincipalName $appID -PermissionsToSecrets Set, Get
```

Para ver otras formas de crear un almacén de Azure Key Vault y almacenar un secreto, consulte los [inicios rápidos de Key Vault](../key-vault/index.yml).

## <a name="import-required-modules-into-your-automation-account"></a>Importación de los módulos necesarios a la cuenta de Automation

Para usar Azure Key Vault en un runbook, debe importar los módulos siguientes en la cuenta de Automation:

* [Az.Accounts](https://www.powershellgallery.com/packages/Az.Accounts)
* [Az.KeyVault](https://www.powershellgallery.com/packages/Az.KeyVault)

Para obtener instrucciones, consulte la sección sobre [importación de módulos de AZ](shared-resources/modules.md#import-az-modules).

## <a name="create-the-runbook-to-send-an-email"></a>Creación del runbook para enviar un correo electrónico

Después de crear un almacén de claves y de almacenar la clave de API de `SendGrid`, es el momento de crear el runbook que recupere la clave de API y envíe un correo electrónico. Vamos a usar un runbook que utiliza `AzureRunAsConnection` como una [cuenta de ejecución](./automation-security-overview.md#run-as-accounts) para autenticarse en Azure con el fin de recuperar el secreto de Azure Key Vault. Llamaremos al runbook **Send-GridMailMessage**. Puede modificar el script de PowerShell utilizado para fines de ejemplo y reutilizarlo para distintos escenarios.

1. Vaya a la cuenta de Azure Automation.
2. En **Automatización de procesos**, seleccione **Runbooks**.
3. En la parte superior de la lista de runbooks, seleccione **+ Crear un runbook**.
4. En la página Agregar Runbook, escriba **Send-GridMailMessage** como nombre del runbook. Como tipo de runbook, seleccione **PowerShell**. Seleccione **Crear**.
   ![Creación del runbook](./media/automation-send-email/automation-send-email-runbook.png)
5. Se crea el runbook y se abre la página Editar Runbook de PowerShell.
   ![Edición del runbook](./media/automation-send-email/automation-send-email-edit.png)
6. Copie el siguiente ejemplo de PowerShell en la página Editar. Asegúrese de que `VaultName` especifique el nombre que haya elegido para la instancia de Key Vault.

    ```powershell-interactive
    Param(
      [Parameter(Mandatory=$True)]
      [String] $destEmailAddress,
      [Parameter(Mandatory=$True)]
      [String] $fromEmailAddress,
      [Parameter(Mandatory=$True)]
      [String] $subject,
      [Parameter(Mandatory=$True)]
      [String] $content
    )

    $Conn = Get-AutomationConnection -Name AzureRunAsConnection
    Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint | Out-Null
    $VaultName = "<Enter your vault name>&quot;
    $SENDGRID_API_KEY = (Get-AzKeyVaultSecret -VaultName $VaultName -Name &quot;SendGridAPIKey").SecretValue
    $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
    $headers.Add("Authorization", "Bearer " + $SENDGRID_API_KEY)
    $headers.Add("Content-Type", "application/json")

    $body = @{
    personalizations = @(
        @{
            to = @(
                    @{
                        email = $destEmailAddress
                    }
            )
        }
    )
    from = @{
        email = $fromEmailAddress
    }
    subject = $subject
    content = @(
        @{
            type = "text/plain"
            value = $content
        }
    )
    }

    $bodyJson = $body | ConvertTo-Json -Depth 4

    $response = Invoke-RestMethod -Uri https://api.sendgrid.com/v3/mail/send -Method Post -Headers $headers -Body $bodyJson
    ```

7. Seleccione **Publicar** para guardar y publicar el runbook.

Para comprobar que el runbook se ejecuta correctamente, puede seguir los pasos descritos en [Prueba de un runbook](manage-runbooks.md#test-a-runbook) o de [Inicio de un runbook](start-runbooks.md).

Si no ve inicialmente el correo electrónico de prueba, compruebe las carpetas de **correo no deseado** y de **spam**.

## <a name="clean-up-resources-after-the-email-operation"></a>Limpieza de los recursos después de la operación de correo electrónico

1. Cuando ya no sea necesario el runbook, selecciónelo en la lista de runbooks y haga clic en **Eliminar**.

2. Elimine el almacén de claves mediante el cmdlet [Remove-AzKeyVault](/powershell/module/az.keyvault/remove-azkeyvault).

```azurepowershell-interactive
$VaultName = "<your KeyVault name>"
$ResourceGroupName = "<your ResourceGroup name>"
Remove-AzKeyVault -VaultName $VaultName -ResourceGroupName $ResourceGroupName
```

## <a name="next-steps"></a>Pasos siguientes

* Para enviar datos de trabajo de runbook al área de trabajo de Log Analytics, consulte [Reenvío de datos de un trabajo de Azure Automation a registros de Azure Monitor](automation-manage-send-joblogs-log-analytics.md).
* Para supervisar los registros y las métricas de nivel básico, consulte [Uso de una alerta para desencadenar un runbook de Azure Automation](automation-create-alert-triggered-runbook.md).
* Para corregir los problemas que surgen durante las operaciones de runbook, consulte [Solución de incidencias de runbooks](./troubleshoot/runbooks.md).
