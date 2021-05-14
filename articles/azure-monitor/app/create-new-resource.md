---
title: Creación de un recurso de Azure Application Insights | Microsoft Docs
description: Describe la configuración manual de la supervisión de Application Insights para una nueva aplicación activa.
ms.topic: conceptual
ms.date: 02/10/2021
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 7e4c4b0ba4dca9b51f120e6092c13195a061aa87
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109754540"
---
# <a name="create-an-application-insights-resource"></a>Creación de recursos en Application Insights

Azure Application Insights muestra los datos de la aplicación en un *recurso* de Microsoft Azure. Por tanto, la creación de un nuevo recurso forma parte de la [configuración de Application Insights para supervisar una aplicación nueva][start]. Después de haber creado el recurso nuevo, puede obtener su clave de instrumentación y usarla para configurar el SDK de Application Insights. La clave de instrumentación vincula la telemetría al recurso.

> [!IMPORTANT]
> [La versión clásica de Application Insights está en desuso](https://azure.microsoft.com/updates/we-re-retiring-classic-application-insights-on-29-february-2024/). Siga estas [instrucciones sobre cómo actualizar a Application Insights basado en áreas de trabajo](convert-classic-resource.md).

## <a name="sign-in-to-microsoft-azure"></a>Iniciar sesión en Microsoft Azure

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="create-an-application-insights-resource"></a>Creación de recursos en Application Insights

Inicie sesión en [Azure Portal](https://portal.azure.com) y cree un recurso de Application Insights:

![En la esquina superior izquierda, haga clic en el signo "+". Seleccionar Herramientas de desarrollo y, a continuación, Application Insights](./media/create-new-resource/new-app-insights.png)

   | Configuración        |  Value           | Descripción  |
   | ------------- |:-------------|:-----|
   | **Nombre**      | `Unique value` | Nombre que identifica la aplicación que está supervisando. |
   | **Grupo de recursos**     | `myResourceGroup`      | Nombre para el grupo de recursos nuevo o existente que hospedará los datos de Application Insights. |
   | **Región** | `East US` | Elija una ubicación cerca de usted o de donde se hospeda la aplicación. |
   | **Modo de recursos** | `Classic` o `Workspace-based` | Los recursos basados en el área de trabajo permiten enviar la telemetría de Application Insights a un área de trabajo común de Log Analytics. Para obtener más información, consulte el [artículo sobre los recursos basados en áreas de trabajo](create-workspace-resource.md).

> [!NOTE]
> Aunque puede usar el mismo nombre de recurso en distintos grupos de recursos, puede ser beneficioso usar un nombre único global. Esto puede ser útil si tiene previsto [realizar consultas entre recursos](../logs/cross-workspace-query.md#identifying-an-application) ya que simplifica la sintaxis necesaria.

Escriba los valores apropiados en los campos obligatorios y, a continuación, seleccione **Revisar y crear**.

> [!div class="mx-imgBorder"]
> ![Escriba los valores en los campos obligatorios y, a continuación, seleccione "Revisar y crear".](./media/create-new-resource/review-create.png)

Cuando se haya creado la aplicación, se abrirá un panel nuevo. En este panel podrá ver los datos de uso y rendimiento de la aplicación supervisada. 

## <a name="copy-the-instrumentation-key"></a>Copia de la clave de instrumentación

La clave de instrumentación identifica el recurso con el que quiere asociar los datos de telemetría. Necesitará copiar la clave de instrumentación y agregarla al código de la aplicación.

> [!IMPORTANT]
> Las nuevas regiones de Azure **requieren** el uso de cadenas de conexión en lugar de claves de instrumentación. La [cadena de conexión](./sdk-connection-string.md?tabs=net) identifica el recurso con el que se quieren asociar los datos de telemetría. También permite modificar los puntos de conexión que va a usar el recurso como destino de la telemetría. Tiene que copiar la cadena de conexión y agregarla al código de la aplicación o a una variable de entorno.

## <a name="install-the-sdk-in-your-app"></a>Instalación del SDK en la aplicación

Instale el SDK de Application Insights en su aplicación. Este paso depende en gran medida del tipo de aplicación.

Use la clave de instrumentación para configurar [el SDK que instala en la aplicación][start].

El SDK incluye módulos estándar que envían telemetría sin tener que escribir código adicional. Para realizar un seguimiento de las acciones del usuario o diagnosticar los problemas con más detalle, [use la API][api] para enviar su propia telemetría.

## <a name="creating-a-resource-automatically"></a>Creación de un recurso de forma automática

### <a name="powershell"></a>PowerShell

Creación de un recurso de Application Insights

```powershell
New-AzApplicationInsights [-ResourceGroupName] <String> [-Name] <String> [-Location] <String> [-Kind <String>]
 [-Tag <Hashtable>] [-DefaultProfile <IAzureContextContainer>] [-WhatIf] [-Confirm] [<CommonParameters>]
```

#### <a name="example"></a>Ejemplo

```powershell
New-AzApplicationInsights -Kind java -ResourceGroupName testgroup -Name test1027 -location eastus
```
#### <a name="results"></a>Results

```powershell
Id                 : /subscriptions/{subid}/resourceGroups/testgroup/providers/microsoft.insights/components/test1027
ResourceGroupName  : testgroup
Name               : test1027
Kind               : web
Location           : eastus
Type               : microsoft.insights/components
AppId              : 8323fb13-32aa-46af-b467-8355cf4f8f98
ApplicationType    : web
Tags               : {}
CreationDate       : 10/27/2017 4:56:40 PM
FlowType           :
HockeyAppId        :
HockeyAppToken     :
InstrumentationKey : 00000000-aaaa-bbbb-cccc-dddddddddddd
ProvisioningState  : Succeeded
RequestSource      : AzurePowerShell
SamplingPercentage :
TenantId           : {subid}
```

Para obtener la documentación completa de PowerShell para este cmdlet y obtener información sobre cómo recuperar la clave de instrumentación, consulte la [documentación de Azure PowerShell](/powershell/module/az.applicationinsights/new-azapplicationinsights).

### <a name="azure-cli-preview"></a>CLI de Azure (versión preliminar)

Para obtener acceso a los comandos de la CLI de Azure de Application Insights en versión preliminar, primero debe ejecutar:

```azurecli
 az extension add -n application-insights
```

Si no ejecuta el comando `az extension add`, aparece un mensaje de error que indica: `az : ERROR: az monitor: 'app-insights' is not in the 'az monitor' command group. See 'az monitor --help'.`

Ahora puede ejecutar lo siguiente para crear el recurso de Application Insights:

```azurecli
az monitor app-insights component create --app
                                         --location
                                         --resource-group
                                         [--application-type]
                                         [--kind]
                                         [--tags]
```

#### <a name="example"></a>Ejemplo

```azurecli
az monitor app-insights component create --app demoApp --location westus2 --kind web -g demoRg --application-type web
```

#### <a name="results"></a>Results

```azurecli
az monitor app-insights component create --app demoApp --location eastus --kind web -g demoApp  --application-type web
{
  "appId": "87ba512c-e8c9-48d7-b6eb-118d4aee2697",
  "applicationId": "demoApp",
  "applicationType": "web",
  "creationDate": "2019-08-16T18:15:59.740014+00:00",
  "etag": "\"0300edb9-0000-0100-0000-5d56f2e00000\"",
  "flowType": "Bluefield",
  "hockeyAppId": null,
  "hockeyAppToken": null,
  "id": "/subscriptions/{subid}/resourceGroups/demoApp/providers/microsoft.insights/components/demoApp",
  "instrumentationKey": "00000000-aaaa-bbbb-cccc-dddddddddddd",
  "kind": "web",
  "location": "eastus",
  "name": "demoApp",
  "provisioningState": "Succeeded",
  "requestSource": "rest",
  "resourceGroup": "demoApp",
  "samplingPercentage": null,
  "tags": {},
  "tenantId": {tenantID},
  "type": "microsoft.insights/components"
}
```

Para obtener la documentación completa de la CLI de Azure para este comando y obtener información sobre cómo recuperar la clave de instrumentación, consulte la [documentación de la CLI de Azure](/cli/azure/monitor/app-insights/component#az_monitor_app_insights_component_create).

## <a name="next-steps"></a>Pasos siguientes
* [Búsqueda de diagnóstico](./diagnostic-search.md)
* [Exploración de métricas](../essentials/metrics-charts.md)
* [Escribir consultas de Analytics](../logs/log-query-overview.md)

<!--Link references-->

[api]: ./api-custom-events-metrics.md
[diagnostic]: ./diagnostic-search.md
[metrics]: ../essentials/metrics-charts.md
[start]: ./app-insights-overview.md
