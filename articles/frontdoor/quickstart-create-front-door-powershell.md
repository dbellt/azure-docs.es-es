---
title: 'Inicio rápido: Configuración de alta disponibilidad con Azure Front Door: Azure PowerShell'
description: En este inicio rápido se mostrará cómo usar Azure Front Door para crear una aplicación web global de alto rendimiento y alta disponibilidad mediante Azure PowerShell.
services: front-door
documentationcenter: na
author: duongau
ms.author: duau
manager: KumudD
ms.date: 04/19/2021
ms.topic: quickstart
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.custom:
- mode-api
ms.openlocfilehash: 17fa18e1f29622b941c281b9cdce27f6e72eb13a
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107739987"
---
# <a name="quickstart-create-a-front-door-for-a-highly-available-global-web-application-using-azure-powershell"></a>Inicio rápido: Creación de una instancia de Front Door para una aplicación web global de alta disponibilidad mediante Azure PowerShell

El primer paso con Azure Front Door es utilizar Azure PowerShell para crear una aplicación web global de alto rendimiento y alta disponibilidad.

Front Door dirige el tráfico web a recursos concretos de un grupo de back-end. Ha definido el dominio del front-end, agregado recursos a un grupo de back-end y creado una regla de enrutamiento. En este artículo se usa una configuración simple de un grupo de back-end con dos recursos de aplicación web y una regla de enrutamiento única que usa la coincidencia de ruta de acceso predeterminada "/*".

:::image type="content" source="media/quickstart-create-front-door/environment-diagram.png" alt-text="Diagrama del entorno de Front Door con PowerShell." border="false":::

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Azure PowerShell instalado localmente o Azure Cloud Shell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-resource-group"></a>Creación de un grupo de recursos

En Azure, puede asignar recursos relacionados a un grupo de recursos. Puede usar un grupo de recursos existente o crear uno nuevo.

Cree un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup):

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroupFD -Location centralus
```

## <a name="create-two-instances-of-a-web-app"></a>Creación de dos instancias de una aplicación web

Este inicio rápido requiere dos instancias de una aplicación web que se ejecuten en regiones diferentes de Azure. Ambas instancias de la aplicación web se ejecutan en modo activo/activo, por lo que cualquiera de ellas puede asumir el tráfico. Esta configuración difiere de una configuración del modo activo/en espera, en la que una realiza la conmutación por error.

Si aún no tiene ninguna aplicación web, use el siguiente script para configurar dos aplicaciones web de ejemplo.

```azurepowershell-interactive
# Create first web app in Central US region.
$webapp1 = New-AzWebApp `
-Name "WebAppContoso-1" `
-Location centralus `
-ResourceGroupName myResourceGroupFD `
-AppServicePlan myAppServicePlanCentralUS

# Create second web app in South Central US region.
$webapp2 = New-AzWebApp `
-Name "WebAppContoso-2" `
-Location southcentralus `
-ResourceGroupName myResourceGroupFD `
-AppServicePlan myAppServicePlanEastUS
```

## <a name="create-a-front-door"></a>Creación de una instancia de Front Door

En esta sección se detalla cómo crear y configurar los componentes siguientes de Front Door:
    
* Un objeto de front-end contiene el dominio predeterminado de Front Door.
* Un grupo de back-end es un conjunto de servidores back-end equivalentes entre los que Front Door equilibra la carga de las solicitudes de cliente.
* Una regla de enrutamiento asigna el host de front-end y un patrón de ruta de acceso de dirección URL coincidente a un grupo de back-end específico.

### <a name="create-a-frontend-object"></a>Creación de un objeto de front-end

El objeto de front-end configura el nombre de host para Front Door. De forma predeterminada, el nombre de host tendrá un sufijo * *.azurefd.net*.

```azurepowershell-interactive
# Create a unique name
$fdname = "contoso-frontend-$(Get-Random)"

#Create the frontend object
$FrontendEndObject = New-AzFrontDoorFrontendEndpointObject `
-Name "frontendEndpoint1" `
-HostName $fdname".azurefd.net"
```

### <a name="create-the-backend-pool"></a>Creación del grupo de servidores back-end

El grupo de back-end consta de las dos aplicaciones web creadas al principio de esta guía de inicio rápido. La configuración del sondeo de estado y el equilibrio de carga definida en este paso usa valores predeterminados.

```azurepowershell-interactive
# Create backend objects that points to the hostname of the web apps
$backendObject1 = New-AzFrontDoorBackendObject `
-Address $webapp1.DefaultHostName
$backendObject2 = New-AzFrontDoorBackendObject `
-Address $webapp2.DefaultHostName

# Create a health probe object
$HealthProbeObject = New-AzFrontDoorHealthProbeSettingObject `
-Name "HealthProbeSetting"

# Create the load balancing setting object
$LoadBalancingSettingObject = New-AzFrontDoorLoadBalancingSettingObject `
-Name "Loadbalancingsetting" `
-SampleSize "4" `
-SuccessfulSamplesRequired "2" `
-AdditionalLatencyInMilliseconds "0"

# Create a backend pool using the backend objects, health probe, and load balancing settings
$BackendPoolObject = New-AzFrontDoorBackendPoolObject `
-Name "myBackendPool" `
-FrontDoorName $fdname `
-ResourceGroupName myResourceGroupFD `
-Backend $backendObject1,$backendObject2 `
-HealthProbeSettingsName "HealthProbeSetting" `
-LoadBalancingSettingsName "Loadbalancingsetting"
```

### <a name="create-a-routing-rule"></a>Crear una regla de enrutamiento

La regla de enrutamiento asigna el grupo de back-end al dominio de front-end y establece el valor coincidente de la ruta de acceso predeterminada en "/*".

```azurepowershell-interactive
# Create a routing rule mapping the frontend host to the backend pool
$RoutingRuleObject = New-AzFrontDoorRoutingRuleObject `
-Name LocationRule `
-FrontDoorName $fdname `
-ResourceGroupName myResourceGroupFD `
-FrontendEndpointName "frontendEndpoint1" `
-BackendPoolName "myBackendPool" `
-PatternToMatch "/*"
```
### <a name="create-the-front-door"></a>Creación de la instancia de Front Door

Ahora que ha creado los objetos necesarios, cree la instancia de Front Door:

```azurepowershell-interactive
# Creates the Front Door
New-AzFrontDoor `
-Name $fdname `
-ResourceGroupName myResourceGroupFD `
-RoutingRule $RoutingRuleObject `
-BackendPool $BackendPoolObject `
-FrontendEndpoint $FrontendEndObject `
-LoadBalancingSetting $LoadBalancingSettingObject `
-HealthProbeSetting $HealthProbeObject
```

Una vez que la implementación se realiza correctamente, puede probarla mediante los pasos que se describen en la sección siguiente.

## <a name="test-the-front-door"></a>Prueba de Front Door

Ejecute los comandos siguientes para obtener el nombre de host de la instancia de Front Door.

```azurepowershell-interactive
# Gets Front Door in resource group and output the hostname of the frontend domain.
$fd = Get-AzFrontDoor -ResourceGroupName myResourceGroupFD
$fd.FrontendEndpoints[0].Hostname
```

Abra un explorador web y escriba el nombre de host obtenido de los comandos. Front Door dirigirá su solicitud a uno de los recursos de back-end. 

:::image type="content" source="./media/quickstart-create-front-door-powershell/front-door-test-page.png" alt-text="Página de pruebas de Front Door":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado con Front Door, elimine el grupo de recursos. Al eliminar el grupo de recursos, también elimina la instancia de Front Door y todos sus recursos relacionados. 

Para eliminar el grupo de recursos, llame al cmdlet `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroupFD
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado lo siguiente:
* Front Door
* Dos aplicaciones web

Para obtener información sobre cómo agregar un dominio personalizado a Front Door, continúe con los tutoriales de Front Door.

> [!div class="nextstepaction"]
> [Agregar un dominio personalizado](front-door-custom-domain.md)
