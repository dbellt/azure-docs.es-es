---
title: Personalización de reglas mediante PowerShell
titleSuffix: Azure Web Application Firewall
description: En este artículo se proporciona información sobre cómo personalizar las reglas de Firewall de aplicaciones web en Application Gateway con PowerShell.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.date: 11/14/2019
ms.author: victorh
ms.topic: article
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0e2bdb2996c0d6f437cfc7c3e933f1ecfc095895
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110668735"
---
# <a name="customize-web-application-firewall-rules-using-powershell"></a>Personalización de reglas de Firewall de aplicaciones web mediante PowerShell

El Firewall de aplicaciones web (WAF) de Azure Application Gateway proporciona protección a las aplicaciones web. Dicha protección la proporciona Open Web Application Security Project (OWASP) Core Rule Set (CRS). Algunas reglas pueden producir falsos positivos y bloquear el tráfico real. Por este motivo, Application Gateway ofrece la posibilidad de personalizar reglas y grupos de reglas. Para más información sobre reglas y grupos de reglas específicos, consulte [Lista de reglas y grupos de reglas de CRS de Firewall de aplicaciones web](application-gateway-crs-rulegroups-rules.md).

## <a name="view-rule-groups-and-rules"></a>Visualización de reglas y grupos de reglas

Los siguientes ejemplos de código muestran cómo ver las reglas y los grupos de reglas que se pueden configurar en una puerta de enlace de aplicaciones con WAF habilitado.

### <a name="view-rule-groups"></a>Visualización del grupos de reglas

En el ejemplo siguiente se muestra cómo ver grupos de reglas:

```powershell
Get-AzApplicationGatewayAvailableWafRuleSets
```

La siguiente salida es una respuesta truncada del ejemplo anterior:

```
OWASP (Ver. 3.0):

    General:
        Description:

        Rules:
            RuleId     Description
            ------     -----------
            200004     Possible Multipart Unmatched Boundary.

    REQUEST-911-METHOD-ENFORCEMENT:
        Description:

        Rules:
            RuleId     Description
            ------     -----------
            911011     Rule 911011
            911012     Rule 911012
            911100     Method is not allowed by policy
            911013     Rule 911013
            911014     Rule 911014
            911015     Rule 911015
            911016     Rule 911016
            911017     Rule 911017
            911018     Rule 911018

    REQUEST-913-SCANNER-DETECTION:
        Description:

        Rules:
            RuleId     Description
            ------     -----------
            913011     Rule 913011
            913012     Rule 913012
            913100     Found User-Agent associated with security scanner
            913110     Found request header associated with security scanner
            913120     Found request filename/argument associated with security scanner
            913013     Rule 913013
            913014     Rule 913014
            913101     Found User-Agent associated with scripting/generic HTTP client
            913102     Found User-Agent associated with web crawler/bot
            913015     Rule 913015
            913016     Rule 913016
            913017     Rule 913017
            913018     Rule 913018

            ...        ...
```

## <a name="disable-rules"></a>Deshabilitar reglas

En el ejemplo siguiente se deshabilitan las reglas `911011` y `911012` en una puerta de enlace de aplicaciones:

```powershell
$disabledrules=New-AzApplicationGatewayFirewallDisabledRuleGroupConfig -RuleGroupName REQUEST-911-METHOD-ENFORCEMENT -Rules 911011,911012
Set-AzApplicationGatewayWebApplicationFirewallConfiguration -ApplicationGateway $gw -Enabled $true -FirewallMode Detection -RuleSetVersion 3.0 -RuleSetType OWASP -DisabledRuleGroups $disabledrules
Set-AzApplicationGateway -ApplicationGateway $gw
```

## <a name="mandatory-rules"></a>Reglas obligatorias

La lista siguiente contiene las condiciones que provocan que WAF bloquee la solicitud durante el modo de prevención (en modo de detección, se registran como excepciones). Estas condiciones no se pueden configurar o deshabilitar:

* La imposibilidad de analizar el cuerpo de la solicitud tiene como resultado que la solicitud se bloquea, a menos que la inspección del cuerpo esté desactivada (XML, JSON, datos de formulario).
* La longitud de los datos del cuerpo de la solicitud (sin archivos) es mayor que el límite configurado.
* El cuerpo de la solicitud (archivos incluidos) es mayor que el límite.
* Se produjo un error interno en el motor de WAF.

Específico de CRS 3.x:

* La puntuación de anomalías de entrada superó el umbral.

## <a name="next-steps"></a>Pasos siguientes

Después de configurar las reglas deshabilitadas, puede aprender a ver los registros de WAFS. Para más información, consulte [Diagnósticos de Application Gateway](../../application-gateway/application-gateway-diagnostics.md#diagnostic-logging).

[fig1]: ./media/application-gateway-customize-waf-rules-portal/1.png
[1]: ./media/application-gateway-customize-waf-rules-portal/figure1.png
[2]: ./media/application-gateway-customize-waf-rules-portal/figure2.png
[3]: ./media/application-gateway-customize-waf-rules-portal/figure3.png
