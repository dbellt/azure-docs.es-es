---
title: 'Listas de exclusión de Firewall de aplicaciones web en Azure Front Door: Azure Portal'
description: En este artículo se proporciona información sobre la configuración de las listas de exclusión en Azure Front con Azure Portal.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.date: 11/10/2020
ms.author: victorh
ms.topic: conceptual
ms.openlocfilehash: 83baf03c414d9b0f7acb6a93db03794a539a3c58
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107860840"
---
# <a name="web-application-firewall-waf-with-front-door-service-exclusion-lists"></a>Listas de exclusión de Firewall de aplicaciones web (WAF) con Front Door Service 

A veces, el Firewall de aplicaciones web (WAF) podría bloquear una solicitud que quiere permitir para su aplicación. Por ejemplo, Active Directory inserta tokens que se usan para la autenticación. Estos tokens pueden contener caracteres especiales que podrían desencadenar un falso positivo de las reglas de WAF. Las listas de exclusión del WAF le permiten omitir determinados atributos de solicitud de una evaluación del WAF.  Se puede configurar una lista de exclusión con [PowerShell](/powershell/module/az.frontdoor/New-AzFrontDoorWafManagedRuleExclusionObject), la [CLI de Azure](/cli/azure/network/front-door/waf-policy/managed-rules/exclusion#az_network_front_door_waf_policy_managed_rules_exclusion_add), [API REST](/rest/api/frontdoorservice/webapplicationfirewall/policies/createorupdate) o Azure Portal. En el ejemplo siguiente se muestra la configuración de Azure Portal. 
## <a name="configure-exclusion-lists-using-the-azure-portal"></a>Configuración de las listas de exclusión mediante Azure Portal
Es posible acceder a **Administrar las exclusiones** desde el portal de WAF en **Reglas administradas**.

![Administrar las exclusiones](../media/waf-front-door-exclusion/exclusion1.png)
![Administrar exclusion_add](../media/waf-front-door-exclusion/exclusion2.png)

 Una lista de exclusión de ejemplo: ![Administrar exclusion_define](../media/waf-front-door-exclusion/exclusion3.png)

En este ejemplo se excluye el valor en el campo del encabezado *user*. Una solicitud válida puede incluir el campo *user* que contiene una cadena que desencadena una regla de inyección de código SQL. En este caso, puede excluir el parámetro *user* para que la regla de WAF no evalúe nada en el campo.

Los siguientes atributos se pueden agregar a las listas de exclusión por su nombre. Los valores de los campos que se usan no se evalúan con las reglas de WAF, pero sí se evalúan sus nombres. Las listas de exclusión quitan la inspección del valor del campo.

* Nombre del encabezado de la solicitud
* Nombre de la cookie de la solicitud
* Nombre de los argumentos de la cadena de consulta
* Nombre de los argumentos de publicación del cuerpo de la solicitud

Puede especificar una coincidencia exacta con un atributo de cadena de consulta, una cookie, el cuerpo o el encabezado de una solicitud.  Si lo desea, también puede especificar coincidencias parciales. Los operadores siguientes son los criterios de coincidencia admitidos:

- **Equals** (es igual a):  este operador se usa para una coincidencia exacta. Por ejemplo, para seleccionar el encabezado denominado **bearerToken**, utilice el operador de igualdad con el selector definido como **bearerToken**.
- **Starts with** (empieza por): este operador coincide con todos los campos que comienzan por el valor del selector especificado.
- **Ends with** (termina en):  este operador coincide con todos los campos de solicitud que terminan con el valor del selector especificado.
- **Contains** (contiene): este operador coincide con todos los campos de solicitud que contienen el valor del selector especificado.
- **Equals any** (es igual a cualquiera): este operador coincide con todos los campos de solicitud. * es el valor del selector.

Los nombres de encabezado y cookies no distinguen mayúsculas de minúsculas.

Si un valor de encabezado, de cookie, de argumento de publicación o de argumento de consulta genera falsos positivos para algunas reglas, puede excluir esa parte de la solicitud para que la regla no la considere:


|matchVariableName de registros de WAF  |Exclusión de reglas en el portal  |
|---------|---------|
|CookieValue:SOME_NAME        |El nombre de la cookie de solicitud es igual a SOME_NAME|
|HeaderValue:SOME_NAME        |El nombre del encabezado de solicitud es igual a SOME_NAME|
|PostParamValue:SOME_NAME     |El nombre de los argumentos del cuerpo de solicitud es igual a SOME_NAME|
|QueryParamValue:SOME_NAME    |El nombre de los argumentos de la cadena de consulta es igual a SOME_NAME|


Actualmente solo se admiten las exclusiones de reglas para los valores de matchVariableNames anteriores en los registros de WAF. Para cualquier otro valor de matchVariableNames, debe deshabilitar las reglas que devuelven falsos positivos, o bien crear una regla personalizada que permita explícitamente esas solicitudes. En concreto, cuando matchVariableName es CookieName, HeaderName, PostParamName o QueryParamName, significa que el propio nombre está desencadenando la regla. La exclusión de reglas no es compatible con estos valores de matchVariableNames en este momento.


Si excluye un argumento POST del cuerpo de la solicitud denominado *FOO*, ninguna regla debe mostrar PostParamValue:FOO como matchVariableName en los registros de WAF. Sin embargo, es posible que aún se muestre una regla con un valor matchVariableName de InitialBodyContents que coincida con el valor del parámetro de envío FOO, ya que los valores del parámetro POST forman parte de InitialBodyContents.

Puede aplicar las listas de exclusión a todas las reglas del conjunto de reglas administradas, a las reglas de un grupo de reglas específico o a una sola regla, tal como se muestra en el ejemplo anterior.

## <a name="define-exclusion-based-on-web-application-firewall-logs"></a>Definición de la exclusión basada en registros de Web Application Firewall
 La [supervisión y registro de Azure Web Application Firewall](waf-front-door-monitor.md) muestra los detalles coincidentes de una solicitud bloqueada. Si un valor de encabezado, de cookie, de argumento de publicación o de argumento de consulta genera falsos positivos para algunas reglas, puede excluir esa parte de la solicitud para que la regla la considere. En la siguiente tabla se muestran los valores de ejemplo de los registros de WAF y las condiciones de exclusión correspondientes.

|matchVariableName de registros de WAF    |Exclusión de reglas en el portal|
|--------|------|
|CookieValue:SOME_NAME  |El nombre de la cookie de solicitud es igual a SOME_NAME|
|HeaderValue:SOME_NAME  |El nombre del encabezado de solicitud es igual a SOME_NAME|
|PostParamValue:SOME_NAME|  El nombre de los argumentos del cuerpo de solicitud es igual a SOME_NAME|
|QueryParamValue:SOME_NAME| El nombre de los argumentos de la cadena de consulta es igual a SOME_NAME|


## <a name="next-steps"></a>Pasos siguientes

Después de configurar el WAF, aprenda a ver los registros del WAF. Para más información, consulte el artículo sobre [diagnósticos de Front Door](../afds/waf-front-door-monitor.md).