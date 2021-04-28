---
title: Registros de ejecución de secuencias en Azure Functions
description: De 115 a 145 caracteres, incluidos los espacios. Este resumen se muestra en el resultado de la búsqueda.
ms.date: 9/1/2020
ms.topic: how-to
ms.custom: contperf-fy21q2, devx-track-azurepowershell
ms.openlocfilehash: cc12af12fa41d5556f2c5a0fc000abbc9c13a0d8
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108076930"
---
# <a name="enable-streaming-execution-logs-in-azure-functions"></a>Habilitación de los registros de ejecución de secuencias en Azure Functions

A menudo, cuando desarrolla una aplicación, quiere ver lo que se escribe en los registros casi en tiempo real cuando se ejecuta en Azure.

Hay dos maneras de ver una secuencia de los archivos de registro que generan las ejecuciones de la función.

* **Streaming integrado de registros**: la plataforma de App Service le permite ver una secuencia de los archivos de registro de aplicaciones. Esto equivale a la salida que se ve al depurar las funciones durante el [desarrollo local](functions-develop-local.md) y cuando se usa la pestaña **Prueba** del portal. Se muestra toda la información basada en el registro. Para más información, consulte [Registros de Stream](../app-service/troubleshoot-diagnostic-logs.md#stream-logs). Este método de streaming solo admite una instancia y no se puede usar con una aplicación que se ejecuta en Linux en un plan de consumo.

* **Live Metrics Stream**: cuando la aplicación de función está [conectada a Application Insights](configure-monitoring.md#enable-application-insights-integration), puede ver los datos de registro y otras métricas casi en tiempo real en Azure Portal mediante [Live Metrics Stream](../azure-monitor/app/live-stream.md). Use este método cuando supervise las funciones que se ejecutan en varias instancias o en Linux en un plan de consumo. En este método se usan [datos muestreados](configure-monitoring.md#configure-sampling).

Las secuencias de registro se pueden ver tanto en el portal como en la mayoría de los entornos de desarrollo locales. 

## <a name="portal"></a>Portal

Puede ver ambos tipos de secuencias de registro en el portal.

### <a name="built-in-log-streaming"></a>Streaming integrado de registros

Para ver los registros de secuencias en el portal, seleccione la pestaña **Características de la plataforma** en la aplicación de función. Luego, en **Supervisión**, elija **Secuencias de registro**.

![Habilitar los registros de secuencias en el portal](./media/functions-monitoring/enable-streaming-logs-portal.png)

Esto conecta la aplicación con el servicio de secuencias de registro, y los registros de aplicación se muestran en la ventana. Puede alternar entre **Registros de aplicaciones** y **Registros de servidor web**.  

![Ver los registros de secuencias en el portal](./media/functions-monitoring/streaming-logs-window.png)

### <a name="live-metrics-stream"></a>Secuencia de métricas en directo

Para ver Live Metrics Stream de su aplicación, seleccione la pestaña **Información general** de la aplicación de funciones. Cuando haya habilitado Application Insights, verá un vínculo a **Application Insights** en **Características configuradas**. Este vínculo lo llevará a la página Application Insights correspondiente a la aplicación.

En Application Insights, seleccione **Live Metrics Stream**. Las [entradas de los registros muestreados](configure-monitoring.md#configure-sampling) se muestran en **Telemetría de ejemplo**.

![Vista de Live Metrics Stream en el portal](./media/functions-monitoring/live-metrics-stream.png) 

## <a name="visual-studio-code"></a>Visual Studio Code

[!INCLUDE [functions-enable-log-stream-vs-code](../../includes/functions-enable-log-stream-vs-code.md)]

## <a name="core-tools"></a>Core Tools

[!INCLUDE [functions-streaming-logs-core-tools](../../includes/functions-streaming-logs-core-tools.md)]

## <a name="azure-cli"></a>Azure CLI

Puede habilitar los registros de secuencias con la [CLI de Azure](/cli/azure/install-azure-cli). Use los siguientes comandos para iniciar sesión, elija su suscripción y transmita los archivos de registro:

```azurecli
az login
az account list
az account set --subscription <subscriptionNameOrId>
az webapp log tail --resource-group <RESOURCE_GROUP_NAME> --name <FUNCTION_APP_NAME>
```

## <a name="azure-powershell"></a>Azure PowerShell

Puede habilitar los registros de secuencias con [Azure PowerShell](/powershell/azure/). Para PowerShell, use el comando [Set-AzWebApp](/powershell/module/az.websites/set-azwebapp) para habilitar el registro en la aplicación de funciones, como se muestra en el siguiente fragmento de código: 

:::code language="powershell" source="~/powershell_scripts/app-service/monitor-with-logs/monitor-with-logs.ps1" range="19-20":::

Para obtener más información, vea el [código de ejemplo completo](../app-service/scripts/powershell-monitor.md#sample-script). 

## <a name="next-steps"></a>Pasos siguientes

+ [Monitor Azure Functions](functions-monitoring.md)
+ [Análisis de la telemetría de Azure Functions en Application Insights](analyze-telemetry-data.md)
