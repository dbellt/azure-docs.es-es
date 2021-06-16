---
title: 'Tutorial: Configuración de la directiva de firewall de aplicaciones web de filtrado geográfico para Azure Front Door'
description: En este tutorial aprenderá a crear una directiva WAF de filtrado geográfico y a asociarla con el host existente de front-end de Front Door.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/14/2020
ms.author: duau
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 818cab91b78d765e6169e9e5a900fc064f71af1f
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110701726"
---
# <a name="tutorial-how-to-set-up-a-geo-filtering-waf-policy-for-your-front-door"></a>Tutorial: Procedimientos para configurar una directiva de WAF de filtrado geográfico para Front Door
En este tutorial se muestra cómo usar Azure PowerShell para crear una directiva de filtrado geográfico de ejemplo y a asociarla con el host de front-end de Front Door existente. Esta directiva de filtrado geográfico de ejemplo bloqueará las solicitudes de todos los países o regiones, salvo Estados Unidos.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> - Definir la condición de coincidencia del filtrado geográfico.
> - Agregar una condición de coincidencia de filtrado geográfico a una regla.
> - Agregar reglas a una directiva.
> - Vincular una directiva de WAF a un host de front-end de Front Door.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos
* Antes de empezar a configurar una directiva de filtrado geográfico, configure el entorno de PowerShell y crear un perfil de Front Door.
* Cree un perfil de Front Door según las instrucciones que se describen en [Inicio rápido: Creación de un perfil de Front Door](quickstart-create-front-door.md).

## <a name="define-geo-filtering-match-condition"></a>Definición de la condición de coincidencia del filtrado geográfico

Cree una condición de coincidencia de ejemplo que selecciona las solicitudes que no proceden de "EE.UU.", utilizando [New AzFrontDoorWafMatchConditionObject](/powershell/module/az.frontdoor/new-azfrontdoorwafmatchconditionobject) en los parámetros al crear una condición de coincidencia. [Aquí](front-door-geo-filtering.md) se proporcionan códigos de país o región de dos letras para la asignación de país o región.

```azurepowershell-interactive
$nonUSGeoMatchCondition = New-AzFrontDoorWafMatchConditionObject `
-MatchVariable RemoteAddr `
-OperatorProperty GeoMatch `
-NegateCondition $true `
-MatchValue "US"
```
## <a name="add-geo-filtering-match-condition-to-a-rule-with-action-and-priority"></a>Adición de una condición de coincidencia de filtrado geográfico a una regla con Action y Priority

Cree un objeto CustomRule `nonUSBlockRule` en función de la condición de coincidencia, un valor de Action y un valor de Priority, para lo que debe usar [New-AzFrontDoorWafCustomRuleObject](/powershell/module/az.frontdoor/new-azfrontdoorwafcustomruleobject).  Un objeto CustomRule puede tener varios MatchCondition.  En este ejemplo, el valor de Action es Block y el de Priority es 1, la prioridad más alta.

```
$nonUSBlockRule = New-AzFrontDoorWafCustomRuleObject `
-Name "geoFilterRule" `
-RuleType MatchRule `
-MatchCondition $nonUSGeoMatchCondition `
-Action Block `
-Priority 1
```
## <a name="add-rules-to-a-policy"></a>Adición de reglas a una directiva
Busque el nombre del grupo de recursos que contiene el perfil de Front Door que usan `Get-AzResourceGroup`. A continuación, cree un objeto de directiva `geoPolicy` que contenga `nonUSBlockRule` para lo que debe usar [New-AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy) en el grupo de recursos especificado que contiene el perfil de Front Door. Debe especificar un nombre único para la directiva de filtrado geográfico. 

En el ejemplo siguiente se usa el nombre de grupo de recursos *FrontDoorQS_rg0* con la suposición de que ha creado el perfil de Front Door mediante las instrucciones proporcionadas en el artículo [Inicio rápido: Creación de una instancia de Front Door](quickstart-create-front-door.md). En el ejemplo siguiente, reemplace el nombre de la directiva *geoPolicyAllowUSOnly* con un nombre de directiva único.

```
$geoPolicy = New-AzFrontDoorWafPolicy `
-Name "geoPolicyAllowUSOnly" `
-resourceGroupName FrontDoorQS_rg0 `
-Customrule $nonUSBlockRule  `
-Mode Prevention `
-EnabledState Enabled
```
## <a name="link-waf-policy-to-a-front-door-frontend-host"></a>Vinculación de una directiva de WAF a un host de front-end de Front Door
Vincule el objeto de directiva de WAF al host de front-end de Front Door existente y actualice las propiedades de Front Door. 

Para ello, en primer lugar recupere el objeto Front Door mediante [Get-AzFrontDoor](/powershell/module/az.frontdoor/get-azfrontdoor). 

```
$geoFrontDoorObjectExample = Get-AzFrontDoor -ResourceGroupName FrontDoorQS_rg0
$geoFrontDoorObjectExample[0].FrontendEndpoints[0].WebApplicationFirewallPolicyLink = $geoPolicy.Id
```
Luego, establezca la propiedad WebApplicationFirewallPolicyLink de front-end en el valor de resourceId de `geoPolicy`mediante [Set-AzFrontDoor](/powershell/module/az.frontdoor/set-azfrontdoor).

```
Set-AzFrontDoor -InputObject $geoFrontDoorObjectExample[0]
```

> [!NOTE] 
> La propiedad WebApplicationFirewallPolicyLink solo es preciso establecerse una vez para vincular una directiva de WAF a un host de front-end de Front Door. Las posteriores actualizaciones de la directiva se aplican al host del front-end.

## <a name="clean-up-resources"></a>Limpieza de recursos

En los pasos anteriores, ha configurado una regla de filtrado geográfico que está asociada a una directiva de WAF. A continuación, ha vinculado la directiva a un host de front-end de Front Door. Si ya no necesita la regla de filtrado geográfico o la directiva de WAF, primero debe desasociar la directiva del host de front-end antes de poder eliminar la directiva de WAF.

:::image type="content" source="media/front-door-geo-filtering/front-door-disassociate-policy.png" alt-text="Desasociar directiva de WAF":::

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo configurar un firewall de aplicaciones web para Front Door, continúe con el siguiente tutorial.

> [!div class="nextstepaction"]
> [Firewall de aplicaciones web y Front Door](front-door-waf.md)
