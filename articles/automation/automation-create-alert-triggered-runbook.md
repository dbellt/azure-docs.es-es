---
title: Uso de una alerta para desencadenar un runbook de Azure Automation
description: En este artículo se indica cómo desencadenar un runbook para que se ejecute cuando se genere una alerta de Azure.
services: automation
ms.subservice: process-automation
ms.date: 02/14/2021
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 36f1881ddd10498e7736de2d117a42021075abdc
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107834883"
---
# <a name="use-an-alert-to-trigger-an-azure-automation-runbook"></a>Uso de una alerta para desencadenar un runbook de Azure Automation

Puede usar [Azure Monitor](../azure-monitor/overview.md) para supervisar métricas de nivel básico y registros para la mayoría de los servicios de Azure. Puede llamar a runbooks de Azure Automation mediante [grupos de acciones](../azure-monitor/alerts/action-groups.md) para automatizar tareas basadas en alertas. En este artículo se muestra cómo configurar y ejecutar un runbook mediante alertas.

## <a name="alert-types"></a>Tipos de alerta

Puede usar runbooks de Automation con tres tipos de alerta:

* Alertas comunes
* Alertas de registros de actividad
* Alertas de métricas casi en tiempo real

> [!NOTE]
> Actualmente, el esquema de alerta común normaliza la experiencia de consumo de notificaciones de alerta en Azure. Históricamente, los tres tipos de alerta de Azure (métrica, registro y registro de actividad) han tenido sus propias plantillas de correo electrónico, esquemas de webhook, etc. Para obtener más información, vea [Esquema de alertas comunes](../azure-monitor/alerts/alerts-common-schema.md)

Cuando una alerta llama a un runbook, la llamada real es una solicitud HTTP POST al webhook. El cuerpo de la solicitud POST contiene un objeto con formato JSON que tiene propiedades útiles relacionadas con la alerta. En la tabla siguiente se enumeran vínculos al esquema de carga de cada tipo de alerta:

|Alerta  |Descripción|Esquema de carga  |
|---------|---------|---------|
|[Alerta común](../azure-monitor/alerts/alerts-common-schema.md)|Esquema de alerta común que normaliza la experiencia de consumo de notificaciones de alerta en Azure en la actualidad.|Esquema de carga de alerta común|
|[Alerta de registro de actividad](../azure-monitor/alerts/activity-log-alerts.md)    |Envía una notificación cuando cualquier evento nuevo del registro de actividad de Azure coincide con las condiciones específicas. Por ejemplo, cuando una operación `Delete VM` se produce en **myProductionResourceGroup** o cuando aparece un nuevo evento de Azure Service Health con un estado Activo.| [Esquema de carga de alertas de registros de actividad](../azure-monitor/alerts/activity-log-alerts-webhook.md)        |
|[Alertas de métricas casi en tiempo real](../azure-monitor/alerts/alerts-metric-near-real-time.md)    |Envía una notificación con más rapidez que las alertas de métrica cuando una o varias métricas de nivel de plataforma cumplen las condiciones especificadas. Por ejemplo, cuando el valor de **% de CPU** en una VM es mayor que 90 y el valor de **Entrada de red** es mayor que 500 MB durante los últimos cinco minutos.| [Esquema de carga de alertas de métricas casi en tiempo real](../azure-monitor/alerts/alerts-webhooks.md#payload-schema)          |

Dado que los datos que cada tipo de alerta proporciona son diferentes, cada tipo de alerta se administra de manera diferente. En la siguiente sección, aprenderá a crear un runbook para administrar distintos tipos de alertas.

## <a name="create-a-runbook-to-handle-alerts"></a>Creación de un runbook para administrar las alertas

Para usar Automation con alertas, necesita un runbook que incorpore lógica que administre la carga de JSON de alerta que se pasa al runbook. El siguiente runbook de ejemplo se debe llamar desde una alerta de Azure.

Como se ha descrito en la sección anterior, cada tipo de alerta tiene un esquema diferente. El script toma los datos del webhook de una alerta del parámetro de entrada del runbook `WebhookData`. Después, el script evalúa la carga de JSON para determinar qué tipo de alerta se usa.

Este ejemplo usa una alerta de una VM. Los datos de la VM se recuperan de la carga y luego se usa esa información para detener dicha VM. La conexión debe estar configurada en la cuenta de Automation donde se ejecuta el runbook. Al usar alertas para desencadenar runbooks, es importante comprobar el estado de la alerta en el runbook que se desencadena. El runbook se desencadena cada vez que la alerta cambia el estado. Las alertas tienen varios estados, y los dos más comunes son Activado y Resuelto. Compruebe el estado de la lógica del runbook para asegurarse de que no se ejecuta más de una vez. En el ejemplo de este artículo se muestra cómo buscar únicamente alertas con el estado Activado.

El runbook usa el activo de conexión `AzureRunAsConnection` [Ejecutar como cuenta](./automation-security-overview.md) para autenticarse en Azure con el fin de realizar la acción de administración en la máquina virtual.

Use este ejemplo para crear un runbook llamado **AzureVmInResponsetoVMAlert Stop**. Puede modificar el script de PowerShell y usarlo con muchos recursos diferentes.

1. Vaya a la cuenta de Azure Automation.
2. En **Automatización de procesos**, seleccione **Runbooks**.
3. En la parte superior de la lista de runbooks, seleccione **+ Crear un runbook**.
4. En la página **Agregar Runbook**, escriba **Stop-AzureVmInResponsetoVMAlert** como nombre del runbook. Como tipo de runbook, seleccione **PowerShell**. Seleccione **Crear**.  
5. Copie el siguiente ejemplo de PowerShell en la página **Editar**.

    ```powershell-interactive
    [OutputType("PSAzureOperationResponse")]
    param
    (
        [Parameter (Mandatory=$false)]
        [object] $WebhookData
    )
    $ErrorActionPreference = "stop"

    if ($WebhookData)
    {
        # Get the data object from WebhookData
        $WebhookBody = (ConvertFrom-Json -InputObject $WebhookData.RequestBody)

        # Get the info needed to identify the VM (depends on the payload schema)
        $schemaId = $WebhookBody.schemaId
        Write-Verbose "schemaId: $schemaId" -Verbose
        if ($schemaId -eq "azureMonitorCommonAlertSchema") {
            # This is the common Metric Alert schema (released March 2019)
            $Essentials = [object] ($WebhookBody.data).essentials
            # Get the first target only as this script doesn't handle multiple
            $alertTargetIdArray = (($Essentials.alertTargetIds)[0]).Split("/")
            $SubId = ($alertTargetIdArray)[2]
            $ResourceGroupName = ($alertTargetIdArray)[4]
            $ResourceType = ($alertTargetIdArray)[6] + "/" + ($alertTargetIdArray)[7]
            $ResourceName = ($alertTargetIdArray)[-1]
            $status = $Essentials.monitorCondition
        }
        elseif ($schemaId -eq "AzureMonitorMetricAlert") {
            # This is the near-real-time Metric Alert schema
            $AlertContext = [object] ($WebhookBody.data).context
            $SubId = $AlertContext.subscriptionId
            $ResourceGroupName = $AlertContext.resourceGroupName
            $ResourceType = $AlertContext.resourceType
            $ResourceName = $AlertContext.resourceName
            $status = ($WebhookBody.data).status
        }
        elseif ($schemaId -eq "Microsoft.Insights/activityLogs") {
            # This is the Activity Log Alert schema
            $AlertContext = [object] (($WebhookBody.data).context).activityLog
            $SubId = $AlertContext.subscriptionId
            $ResourceGroupName = $AlertContext.resourceGroupName
            $ResourceType = $AlertContext.resourceType
            $ResourceName = (($AlertContext.resourceId).Split("/"))[-1]
            $status = ($WebhookBody.data).status
        }
        elseif ($schemaId -eq $null) {
            # This is the original Metric Alert schema
            $AlertContext = [object] $WebhookBody.context
            $SubId = $AlertContext.subscriptionId
            $ResourceGroupName = $AlertContext.resourceGroupName
            $ResourceType = $AlertContext.resourceType
            $ResourceName = $AlertContext.resourceName
            $status = $WebhookBody.status
        }
        else {
            # Schema not supported
            Write-Error "The alert data schema - $schemaId - is not supported."
        }

        Write-Verbose "status: $status" -Verbose
        if (($status -eq "Activated") -or ($status -eq "Fired"))
        {
            Write-Verbose "resourceType: $ResourceType" -Verbose
            Write-Verbose "resourceName: $ResourceName" -Verbose
            Write-Verbose "resourceGroupName: $ResourceGroupName" -Verbose
            Write-Verbose "subscriptionId: $SubId" -Verbose

            # Determine code path depending on the resourceType
            if ($ResourceType -eq "Microsoft.Compute/virtualMachines")
            {
                # This is an Resource Manager VM
                Write-Verbose "This is an Resource Manager VM." -Verbose

                # Authenticate to Azure with service principal and certificate and set subscription
                Write-Verbose "Authenticating to Azure with service principal and certificate" -Verbose
                $ConnectionAssetName = "AzureRunAsConnection"
                Write-Verbose "Get connection asset: $ConnectionAssetName" -Verbose
                $Conn = Get-AutomationConnection -Name $ConnectionAssetName
                if ($Conn -eq $null)
                {
                    throw "Could not retrieve connection asset: $ConnectionAssetName. Check that this asset exists in the Automation account."
                }
                Write-Verbose "Authenticating to Azure with service principal." -Verbose
                Add-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint | Write-Verbose
                Write-Verbose "Setting subscription to work against: $SubId" -Verbose
                Set-AzContext -SubscriptionId $SubId -ErrorAction Stop | Write-Verbose

                # Stop the Resource Manager VM
                Write-Verbose "Stopping the VM - $ResourceName - in resource group - $ResourceGroupName -" -Verbose
                Stop-AzVM -Name $ResourceName -ResourceGroupName $ResourceGroupName -Force
                # [OutputType(PSAzureOperationResponse")]
            }
            else {
                # ResourceType not supported
                Write-Error "$ResourceType is not a supported resource type for this runbook."
            }
        }
        else {
            # The alert status was not 'Activated' or 'Fired' so no action taken
            Write-Verbose ("No action taken. Alert status: " + $status) -Verbose
        }
    }
    else {
        # Error
        Write-Error "This runbook is meant to be started from an Azure alert webhook only."
    }
    ```

6. Seleccione **Publicar** para guardar y publicar el runbook.

## <a name="create-the-alert"></a>Crear la alerta

Las alertas usan grupos de acciones, que son colecciones de acciones que la alerta desencadena. Los runbooks son simplemente una de las muchas acciones que puede usar con los grupos de acciones.

1. En la cuenta de Automation, seleccione **Alertas** en **Supervisión**.
1. Seleccione **+ Nueva regla de alertas**.
1. Haga clic en **Seleccionar** en **Recurso**. En la página **Seleccionar un recurso**, seleccione la máquina virtual a la que alertar y haga clic en **Listo**.
1. Haga clic en **Agregar condición** en **Condición**. Seleccione la señal que quiere usar, por ejemplo, **Porcentaje de CPU**, y haga clic en **Listo**.
1. En la página **Configurar lógica de señal**, escriba el **Valor de umbral** en **Lógica de alerta** y haga clic en **Listo**.
1. En **Grupos de acciones**, seleccione **Crear nuevo**.
1. En la página **Agregar grupo de acciones**, asigne un nombre y un nombre corto al grupo de acciones.
1. Asigne un nombre a la acción. Como tipo de acción, seleccione **Runbook de Automation**.
1. Seleccione **Editar detalles**. En la página **Configurar Runbook**, en **Origen de runbook**, seleccione **Usuario**.  
1. Seleccione su **Suscripción** y **Cuenta de Automation** y luego seleccione el runbook **AzureVmInResponsetoVMAlert Stop**.  
1. Seleccione **Sí** en **Habilitar el esquema de alerta común**.
1. Para crear el grupo de acciones, seleccione **Aceptar**.

    ![Página Agregar grupo de acciones](./media/automation-create-alert-triggered-runbook/add-action-group.png)

    Puede usar este grupo de acciones en las [alertas de registro de actividad](../azure-monitor/alerts/activity-log-alerts.md) y las [alertas casi en tiempo real](../azure-monitor/alerts/alerts-overview.md) que cree.

1. En **Detalles de alertas**, agregue un nombre de regla de alerta y una descripción y haga clic en **Crear regla de alerta**.

## <a name="next-steps"></a>Pasos siguientes

* Para iniciar un runbook con un webhook, vea [Iniciar un runbook desde un webhook](automation-webhooks.md).
* Para descubrir diferentes maneras de iniciar un runbook, vea [Iniciar un runbook](./start-runbooks.md).
* Para crear una alerta de registro de actividad, vea [Creación de alertas del registro de actividad](../azure-monitor/alerts/activity-log-alerts.md).
* Para aprender a crear una alerta casi en tiempo real, consulte [Creación de una regla de alertas en Azure Portal](../azure-monitor/alerts/alerts-metric.md?toc=/azure/azure-monitor/toc.json).
* Para obtener una referencia de los cmdlets de PowerShell, consulte [Az.Automation](/powershell/module/az.automation).
