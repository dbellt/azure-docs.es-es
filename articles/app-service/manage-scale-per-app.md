---
title: Escalado por aplicación para el hospedaje de alta densidad
description: Escale aplicaciones independientemente de los planes de App Service y optimice las instancias escaladas horizontalmente en el plan.
author: btardif
ms.assetid: a903cb78-4927-47b0-8427-56412c4e3e64
ms.topic: article
ms.date: 05/13/2019
ms.author: byvinyal
ms.custom: seodec18, devx-track-azurepowershell
ms.openlocfilehash: 756117a2a231fcb406fd3e3102a16c318c621aa0
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107832615"
---
# <a name="high-density-hosting-on-azure-app-service-using-per-app-scaling"></a>Hospedaje de alta densidad en Azure App Service con escalado por aplicación

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Al usar App Service, puede escalar sus aplicaciones si escala el [plan de App Service](overview-hosting-plans.md) en el que se ejecutan. Cuando se ejecutan varias aplicaciones en el mismo plan de App Service, cada instancia escalada horizontalmente ejecuta todas las aplicaciones del plan.

El *escalado por aplicación* puede habilitarse en el nivel del plan de App Service para permitir el escalado de una aplicación independientemente del plan de App Service que la hospeda. De esta manera, se puede escalar un plan de App Service a 10 instancias, pero se puede configurar una aplicación para usar solo cinco.

> [!NOTE]
> El escalado por aplicación solo está disponible para los planes de tarifa **Estándar**, **Premium**, **Premium V2** y **Aislado**.
>

A las aplicaciones se les asigna un plan de App Service disponible de la mejor manera posible para lograr una distribución uniforme entre las instancias. Aunque no es posible garantizar una distribución uniforme, la plataforma sí se asegurará de que dos instancias de la misma aplicación no se hospeden en la misma instancia del plan de App Service.

La plataforma no se basa en métricas para decidir la asignación de trabajo. Las aplicaciones se vuelven a equilibrar solo cuando se agregan o quitan instancias en el plan de App Service.

## <a name="per-app-scaling-using-powershell"></a>Escalado por aplicación mediante PowerShell

Cree un plan con escalado por aplicación; para ello, pase el parámetro ```-PerSiteScaling $true``` al cmdlet ```New-AzAppServicePlan```.

```powershell
New-AzAppServicePlan -ResourceGroupName $ResourceGroup -Name $AppServicePlan `
                            -Location $Location `
                            -Tier Premium -WorkerSize Small `
                            -NumberofWorkers 5 -PerSiteScaling $true
```

Habilite el escalado por aplicación con un plan de App Service existente; para ello, pase el parámetro `-PerSiteScaling $true` al cmdlet ```Set-AzAppServicePlan```.

```powershell
# Enable per-app scaling for the App Service Plan using the "PerSiteScaling" parameter.
Set-AzAppServicePlan -ResourceGroupName $ResourceGroup `
   -Name $AppServicePlan -PerSiteScaling $true
```

En el nivel de aplicación, configure el número de instancias que puede usar la aplicación en el plan de App Service.

En el ejemplo siguiente, la aplicación está limitada a dos instancias, independientemente de la cantidad de estas a las que se escale horizontalmente el plan de App Service subyacente.

```powershell
# Get the app we want to configure to use "PerSiteScaling"
$newapp = Get-AzWebApp -ResourceGroupName $ResourceGroup -Name $webapp

# Modify the NumberOfWorkers setting to the desired value.
$newapp.SiteConfig.NumberOfWorkers = 2

# Post updated app back to azure
Set-AzWebApp $newapp
```

> [!IMPORTANT]
> `$newapp.SiteConfig.NumberOfWorkers` es diferente de `$newapp.MaxNumberOfWorkers`. En el escalado por aplicación se usa `$newapp.SiteConfig.NumberOfWorkers` para determinar las características de escalado de la aplicación.

## <a name="per-app-scaling-using-azure-resource-manager"></a>Escalado por aplicación mediante Azure Resource Manager

La siguiente plantilla de Azure Resource Manager crea:

- Un plan de App Service que se escala horizontalmente a 10 instancias.
- Una aplicación que está configurada para escalarse hasta un máximo de 5 instancias.

El plan de App Service es establecer la propiedad **PerSiteScaling** en true `"perSiteScaling": true`. La aplicación configura el **número de trabajadores** que se va a usar en 5 `"properties": { "numberOfWorkers": "5" }`.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters":{
        "appServicePlanName": { "type": "string" },
        "appName": { "type": "string" }
        },
    "resources": [
    {
        "comments": "App Service Plan with per site perSiteScaling = true",
        "type": "Microsoft.Web/serverFarms",
        "sku": {
            "name": "P1",
            "tier": "Premium",
            "size": "P1",
            "family": "P",
            "capacity": 10
            },
        "name": "[parameters('appServicePlanName')]",
        "apiVersion": "2015-08-01",
        "location": "West US",
        "properties": {
            "name": "[parameters('appServicePlanName')]",
            "perSiteScaling": true
        }
    },
    {
        "type": "Microsoft.Web/sites",
        "name": "[parameters('appName')]",
        "apiVersion": "2015-08-01-preview",
        "location": "West US",
        "dependsOn": [ "[resourceId('Microsoft.Web/serverFarms', parameters('appServicePlanName'))]" ],
        "properties": { "serverFarmId": "[resourceId('Microsoft.Web/serverFarms', parameters('appServicePlanName'))]" },
        "resources": [ {
                "comments": "",
                "type": "config",
                "name": "web",
                "apiVersion": "2015-08-01",
                "location": "West US",
                "dependsOn": [ "[resourceId('Microsoft.Web/Sites', parameters('appName'))]" ],
                "properties": { "numberOfWorkers": "5" }
            } ]
        }]
}
```

## <a name="recommended-configuration-for-high-density-hosting"></a>Configuración recomendada para el hospedaje de alta densidad

El escalado por aplicación es una característica que está habilitada tanto en las regiones de Azure globales como en los entornos de [App Service Environment](environment/app-service-app-service-environment-intro.md). Sin embargo, la estrategia recomendada es usar entornos de App Service para aprovechar sus características avanzadas y la mayor capacidad del plan de App Service.  

Siga estos pasos para configurar el hospedaje de alta densidad para las aplicaciones:

1. Designe un plan de App Service como plan de alta densidad y escálelo horizontalmente a la capacidad deseada.
1. Establezca la marca `PerSiteScaling` en true en el plan de App Service.
1. Las nuevas aplicaciones se crean y se asignan al plan de App Service con la propiedad **numberOfWorkers** establecida en **1**.
   - El uso de esta configuración produciría la máxima densidad posible.
1. El número de trabajadores se puede configurar por separado por cada aplicación para conceder recursos adicionales según sea necesario. Por ejemplo:
   - En una aplicación de uso elevado, puede establecer **numberOfWorkers** en **3** para que haya una mayor capacidad de procesamiento para esa aplicación.
   - En las aplicaciones de poco uso se establecerá **numberOfWorkers** en **1**.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción detallada sobre los planes de Azure App Service](overview-hosting-plans.md)
- [Introducción al entorno de App Service](environment/app-service-app-service-environment-intro.md)
