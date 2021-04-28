---
title: Ajuste de Web Application Firewall (WAF) para Azure Front Door
description: En este artículo, aprenderá a ajustar WAF para Front Door.
services: web-application-firewall
author: mohitkusecurity
ms.service: web-application-firewall
ms.topic: conceptual
ms.date: 12/11/2020
ms.author: mohitku
ms.reviewer: tyao
ms.openlocfilehash: c0879edc0e3fbd6cf6bcadc26dd862f95ecf4fd4
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107872360"
---
# <a name="tuning-web-application-firewall-waf-for-azure-front-door"></a>Ajuste de Web Application Firewall (WAF) para Azure Front Door
 
El conjunto de reglas predeterminado administrado por Azure se basa en el [conjunto de reglas de núcleo (CRS) de OWASP](https://github.com/SpiderLabs/owasp-modsecurity-crs/tree/v3.1/dev) y está diseñado para ser estricto de manera predeterminada. A menudo se espera que las reglas de WAF deban ajustarse para satisfacer las necesidades específicas de la aplicación u organización mediante WAF. Normalmente esto se logra definiendo exclusiones de reglas, creando reglas personalizadas e incluso deshabilitando reglas que puedan causar problemas o falsos positivos. Hay algunas cosas que puede hacer si se bloquean las solicitudes que se deben pasar a través del firewall de aplicaciones web (WAF).

En primer lugar, asegúrese de que ha leído la [información general de Front Door con WAF](afds-overview.md) y los documentos de la [directiva de WAF para Front Door](waf-front-door-create-portal.md). Asimismo, asegúrese de que ha habilitado la [supervisión y el registro de WAF](waf-front-door-monitor.md). En estos artículos se explica cómo funcionan el firewall de aplicaciones web y los conjuntos de reglas de WAF, y cómo acceder a los registros del firewall de aplicaciones web.
 
## <a name="understanding-waf-logs"></a>Descripción de los registros de WAF
 
El propósito de los registros de WAF es mostrar todas las solicitudes que coinciden con el firewall de aplicaciones web o que ha bloqueado. Es una recopilación de todas las solicitudes evaluadas que coinciden o se han bloqueado. Si observa que el firewall de aplicaciones web bloquea una solicitud que no debería (un falso positivo), puede hacer algunas cosas. En primer lugar, restrinja y busque la solicitud específica. Si lo desea, puede [configurar un mensaje de respuesta personalizado](./waf-front-door-configure-custom-response-code.md) para incluir el campo `trackingReference` a fin de identificar fácilmente el evento y realizar una consulta de registro sobre ese valor específico. Examine los registros para buscar el URI, la marca de tiempo o la IP de cliente específicos de la solicitud. Cuando encuentre las entradas del registro relacionadas, puede empezar a actuar sobre los falsos positivos. 
 
Por ejemplo, imagine que tiene un tráfico legítimo que contiene la cadena `1=1` que quiere pasar a través del WAF. La solicitud tendrá una apariencia similar a la siguiente:

```
POST http://afdwafdemosite.azurefd.net/api/Feedbacks HTTP/1.1
Host: afdwafdemosite.azurefd.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 55

UserId=20&captchaId=7&captchaId=15&comment="1=1"&rating=3
```

Si intenta realizar la solicitud, el firewall de aplicaciones web bloquea el tráfico que contenga la cadena *1=1* en cualquier campo o parámetro. Esta cadena se suele asociar a un ataque por inyección de código SQL. Puede examinar los registros y ver la marca de tiempo de la solicitud y las reglas que se han bloqueado o que coinciden.
 
En el siguiente ejemplo, exploramos un registro `FrontdoorWebApplicationFirewallLog` generado debido a una coincidencia de regla. Se puede usar la siguiente consulta de Log Analytics para buscar las solicitudes que se han bloqueado en las últimas 24 horas:

```kusto
AzureDiagnostics
| where Category == 'FrontdoorWebApplicationFirewallLog'
| where TimeGenerated > ago(1d)
| where action_s == 'Block'

```
 
En el campo `requestUri`, puede ver que la solicitud se realizó específicamente para `/api/Feedbacks/`. Si avanzamos un poco más, encontraremos el identificador de regla `942110` en el campo `ruleName`. Si conoce el identificador de regla, podría ir al [repositorio oficial del conjunto de reglas de núcleo ModSecurity de OWASP](https://github.com/coreruleset/coreruleset) y buscar por ese [identificador de regla](https://github.com/coreruleset/coreruleset/blob/v3.1/dev/rules/REQUEST-942-APPLICATION-ATTACK-SQLI.conf) para revisar su código y saber exactamente en qué coincide esta regla. 
 
A continuación, al comprobar el campo `action`, vemos que esta regla se establece para bloquear solicitudes tras la coincidencia y confirmamos que, de hecho, WAF bloqueó la solicitud al establecerse `policyMode` en `prevention`. 
 
Comprobemos ahora la información en el campo `details`. Aquí puede ver la información de `matchVariableName` y `matchVariableValue`. Sabemos que esta regla se desencadenó porque alguien escribió *1=1* en el campo `comment` de la aplicación web.
 
```json
{
    "time": "2020-09-24T16:43:04.5422943Z",
    "resourceId": "/SUBSCRIPTIONS/<Subscription ID>/RESOURCEGROUPS/<Resource Group Name>/PROVIDERS/MICROSOFT.NETWORK/FRONTDOORS/AFDWAFDEMOSITE",
    "category": "FrontdoorWebApplicationFirewallLog",
    "operationName": "Microsoft.Network/FrontDoor/WebApplicationFirewallLog/Write",
    "properties": {
        "clientIP": "1.1.1.1",
        "clientPort": "53566",
        "socketIP": "1.1.1.1",
        "requestUri": "http://afdwafdemosite.azurefd.net:80/api/Feedbacks/",
        "ruleName": "DefaultRuleSet-1.0-SQLI-942110",
        "policy": "AFDWAFDemoPolicy",
        "action": "Block",
        "host": "afdwafdemosite.azurefd.net",
        "trackingReference": "0mMxsXwAAAABEalekYeI4S55qpi5R7R0/V1NURURHRTA4MTIAZGI4NGQzZDgtNWQ5Ny00ZWRkLTg2ZGYtZDJjNThlMzI2N2I4",
        "policyMode": "prevention",
        "details": {
            "matches": [
                {
                    "matchVariableName": "PostParamValue:comment",
                    "matchVariableValue": "\"1=1\""
                }
            ],
            "msg": "SQL Injection Attack: Common Injection Testing Detected",
            "data": "Matched Data: \"1=1\" found within PostParamValue:comment: \"1=1\""
        }
    }
}
```
 
También hay cierto valor en comprobar los registros de acceso para expandir su conocimiento sobre un evento WAF determinado. A continuación, revisamos el registro `FrontdoorAccessLog` que se generó como respuesta al evento anterior.
 
Puede ver que se trata de registros relacionados basados en el valor `trackingReference` que es el mismo. Entre los diversos campos que proporcionan información general, como `userAgent` y `clientIP`, llamamos la atención sobre los campos `httpStatusCode` y `httpStatusDetails`. Aquí, podemos confirmar que el cliente ha recibido una respuesta HTTP 403, que confirma absolutamente que esta solicitud se denegó y bloqueó. 
 
```json
{
    "time": "2020-09-24T16:43:04.5430764Z",
    "resourceId": "/SUBSCRIPTIONS/<Subscription ID>/RESOURCEGROUPS/<Resource Group Name>/PROVIDERS/MICROSOFT.NETWORK/FRONTDOORS/AFDWAFDEMOSITE",
    "category": "FrontdoorAccessLog",
    "operationName": "Microsoft.Network/FrontDoor/AccessLog/Write",
    "properties": {
        "trackingReference": "0mMxsXwAAAABEalekYeI4S55qpi5R7R0/V1NURURHRTA4MTIAZGI4NGQzZDgtNWQ5Ny00ZWRkLTg2ZGYtZDJjNThlMzI2N2I4",
        "httpMethod": "POST",
        "httpVersion": "1.1",
        "requestUri": "http://afdwafdemosite.azurefd.net:80/api/Feedbacks/",
        "requestBytes": "2160",
        "responseBytes": "324",
        "userAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36",
        "clientIp": "1.1.1.1",
        "socketIp": "1.1.1.1",
        "clientPort": "53566",
        "timeToFirstByte": "0.01",
        "timeTaken": "0.011",
        "securityProtocol": "",
        "routingRuleName": "DemoBERoutingRule",
        "rulesEngineMatchNames": [],
        "backendHostname": "13.88.65.130:3000",
        "isReceivedFromClient": true,
        "httpStatusCode": "403",
        "httpStatusDetails": "403",
        "pop": "WST",
        "cacheStatus": "CONFIG_NOCACHE"
    }
}
```

## <a name="resolving-false-positives"></a>Resolución de falsos positivos
 
Para tomar una decisión informada sobre el control de un falso positivo, es importante familiarizarse con las tecnologías que usa la aplicación. Por ejemplo, imagine que en la pila de tecnología no hay un servidor SQL y que recibe falsos positivos relacionados con esas reglas. La deshabilitación de esas reglas no debilita necesariamente la seguridad. 

Con esta información y sabiendo que la regla 942110 es la que coincide con la cadena `1=1` en nuestro ejemplo, podemos hacer algunas cosas para evitar que se bloquee esta solicitud legítima:
 
* Uso de listas de exclusión
  * Consulte [Web Application Firewall (WAF) con listas de exclusión de Front Door Service](waf-front-door-exclusion.md) para obtener más información sobre las listas de exclusión. 
* Cambio de acciones de WAF
  * Consulte [Acciones de WAF](afds-overview.md#waf-actions) para obtener más información sobre las acciones que se pueden realizar cuando una solicitud coincide con las condiciones de una regla.
* Uso de reglas personalizadas
  * Consulte [Reglas personalizadas para Web Application Firewall con Azure Front Door](waf-front-door-custom-rules.md) para más información acerca de las reglas personalizadas.
* Deshabilitar reglas 

> [!TIP]
> Al seleccionar un enfoque para permitir solicitudes legítimas a través de WAF, trate de acotarlo tanto como pueda. Por ejemplo, es mejor usar una lista de exclusión que deshabilitar una regla por completo.

### <a name="using-exclusion-lists"></a>Uso de listas de exclusión

Una ventaja de usar una lista de exclusión es que solo la variable de coincidencia que seleccione para excluir ya no se inspeccionará para esa solicitud determinada. Es decir, puede elegir entre encabezados de solicitud, cookies de solicitud, argumentos de cadena de consulta o argumentos de publicación del cuerpo de la solicitud específicos que se van a excluir si se cumple una condición determinada, en lugar de evitar la inspección de toda la solicitud. Las demás variables no especificadas de la solicitud seguirán inspeccionándose con normalidad.
 
Es importante tener en cuenta que las exclusiones son una configuración global. Esto significa que la exclusión configurada se aplicará a todo el tráfico que pase a través de WAF, no solo a una aplicación web o un URI específicos. Por ejemplo, esto podría ser un problema si *1=1* es una solicitud válida en el cuerpo de una aplicación web concreta, pero no para otras en la misma directiva de WAF. Si tiene sentido usar diferentes listas de exclusión para diversas aplicaciones, considere la posibilidad de usar distintas directivas de WAF para cada aplicación y aplicarlas al front-end de cada aplicación.
 
Al configurar listas de exclusión para reglas administradas, puede elegir excluir todas las reglas de un conjunto de reglas, todas las reglas de un grupo de reglas o una regla individual. Se puede configurar una lista de exclusión con [PowerShell](/powershell/module/az.frontdoor/New-AzFrontDoorWafManagedRuleExclusionObject), la [CLI de Azure](/cli/azure/network/front-door/waf-policy/managed-rules/exclusion#az_network_front_door_waf_policy_managed_rules_exclusion_add), [API de REST](/rest/api/frontdoorservice/webapplicationfirewall/policies/createorupdate) o Azure Portal.

* Exclusiones en un nivel de regla
  * La aplicación de exclusiones en un nivel de regla significa que las exclusiones especificadas no se analizarán solo en esa regla individual, aunque todas las demás reglas del conjunto de reglas procederán con el análisis. Este es el nivel más granular para las exclusiones y se puede usar para ajustar el conjunto de reglas administrado basado en la información que encuentre en los registros de WAF al solucionar los problemas de un evento.
* Exclusiones en el nivel de grupo de reglas
  * La aplicación de exclusiones en un nivel de grupo de reglas significa que las exclusiones especificadas no se analizarán en ese conjunto específico de tipos de regla. Por ejemplo, seleccionar *SQLI* como un grupo de reglas excluido indica que ninguna de las reglas específicas de SQLI inspeccionaría las exclusiones de solicitudes definidas, pero las reglas de otros grupos, como *PHP*, *RFI* o *XSS* seguirían realizando la inspección. Este tipo de exclusión puede ser útil cuando estamos seguros de que la aplicación no es susceptible a tipos de ataques específicos. Por ejemplo, una aplicación que no tiene ninguna base de datos SQL podría tener todas las reglas *SQLI* excluidas sin que esto sea perjudicial para su nivel de seguridad.
* Exclusiones en el nivel de conjunto de reglas 
  * La aplicación de exclusiones en un nivel de conjunto de reglas significa que las exclusiones especificadas no se analizarán en ninguna de las reglas de seguridad disponibles en ese conjunto de reglas. Se trata de una exclusión completa, por lo que debe usarse con cuidado.

En este ejemplo, realizaremos una exclusión en el nivel más granular (aplicación de la exclusión a una sola regla) y queremos excluir la variable de coincidencia **Nombre de los argumentos de publicación del cuerpo de la solicitud** que contiene `comment`. Esto es evidente, ya que puede ver los detalles de la variable de coincidencia en el registro del firewall: `"matchVariableName": "PostParamValue:comment"`. El atributo es `comment`. También puede encontrar este nombre de atributo de otras formas; vea [Búsqueda de nombres de atributo de solicitud](#finding-request-attribute-names).

![Reglas de exclusión](../media/waf-front-door-tuning/exclusion-rules.png)

![Exclusión de reglas para una regla específica](../media/waf-front-door-tuning/exclusion-rule.png)

En ocasiones, hay casos en los que parámetros específicos se pasan al firewall de aplicaciones web de una forma que puede no ser intuitiva. Por ejemplo, hay un token que se pasa al autenticarse con Azure Active Directory. Este token, `__RequestVerificationToken`, normalmente se pasa como una cookie de solicitud. Pero en algunos casos en los que las cookies están deshabilitadas, este token también se pasa como un argumento POST de solicitud. Por este motivo, para solucionar los falsos positivos del token de Azure AD, debe asegurarse de que `__RequestVerificationToken` se agrega a la lista de exclusión para `RequestCookieNames` y `RequestBodyPostArgsNames`.

Las exclusiones en un nombre de campo (*selector*) significan que WAF dejará de evaluar el valor. Sin embargo, el propio nombre de campo sigue evaluándose y, en raras ocasiones, puede coincidir con una regla de WAF y desencadenar una acción.

![Exclusión de reglas para el conjunto de reglas](../media/waf-front-door-tuning/exclusion-rule-selector.png)

### <a name="changing-waf-actions"></a>Cambio de las acciones de WAF

Otra manera de controlar el comportamiento de las reglas de WAF es elegir la acción que llevará a cabo cuando una solicitud coincida con las condiciones de una regla. Las acciones disponibles son: [Allow, Block, Log y Redirect](afds-overview.md#waf-actions).

En este ejemplo, cambiamos la acción predeterminada *Block* por la acción *Log* en la regla 942110. Esto hará que WAF registre la solicitud y siga evaluando la misma solicitud con respecto a las reglas de prioridad inferior restantes.

![Acciones de WAF](../media/waf-front-door-tuning/actions.png)

Después de realizar la misma solicitud, podemos volver a consultar los registros y veremos que esta solicitud era una coincidencia en el identificador de regla 942110, y que el campo `action_s` ahora indica *Log* en lugar de *Block*. A continuación, ampliamos la consulta de registro para incluir la información de `trackingReference_s` y ver qué más ha ocurrido con esta solicitud.

![Registro que muestra varias coincidencias de regla](../media/waf-front-door-tuning/actions-log.png)

Curiosamente, vemos que una coincidencia de regla SQLI se produce milisegundos después de que se procesara el identificador de regla 942110. La misma solicitud coincidía en el identificador de regla 942310 y esta vez se desencadenó la acción predeterminada *Block*.

Otra ventaja de usar la acción *Log* durante el ajuste o la solución de problemas de WAF es que puede identificar si varias reglas de un grupo de reglas específico coinciden con una solicitud determinada y la bloquean. Después, puede crear las exclusiones en el nivel adecuado, es decir, en el nivel de regla o grupo de reglas. 

### <a name="using-custom-rules"></a>Uso de reglas personalizadas

Una vez que haya identificado lo que causa una coincidencia de regla de WAF, podrá usar reglas personalizadas para ajustar cómo responde WAF al evento. Las reglas personalizadas se procesan antes que las administradas, pueden contener más de una condición y sus acciones pueden ser [Allow, Deny, Log o Redirect](afds-overview.md#waf-actions). Cuando hay una coincidencia de regla, el motor de WAF detiene el procesamiento. Esto significa que ya no se ejecutan otras reglas personalizadas con prioridad más baja ni tampoco reglas administradas.

En el siguiente ejemplo, creamos una regla personalizada con dos condiciones. La primera condición busca el valor `comment` en el cuerpo de la solicitud. La segunda condición busca el valor `/api/Feedbacks/` en el URI de solicitud.

El uso de una regla personalizada le permite ser el más granular al ajustar las reglas de WAF y para lidiar con falsos positivos. En este caso, no realizamos acciones solo en función del valor del cuerpo de la solicitud `comment`, que podría existir en varios sitios o aplicaciones en la misma directiva de WAF. Al incluir otra condición para que coincida también en un URI de solicitud determinado `/api/Feedbacks/`, garantizamos que esta regla personalizada se aplica realmente a este caso de uso explícito que investigamos. Esto garantiza que el motor de WAF aún pueda inspeccionar y evitar el mismo ataque si se realiza en diferentes condiciones.

![Log](../media/waf-front-door-tuning/custom-rule.png)

Al explorar el registro, puede ver que el campo `ruleName_s` contiene el nombre asignado a la regla personalizada que creamos: `redirectcomment`. En el campo `action_s`, puede ver que se realizó la acción *Redirect* para este evento. En el campo `details_matches_s`, podemos ver que se estableció la coincidencia de los detalles de ambas condiciones.

### <a name="disabling-rules"></a>Deshabilitación de reglas

Otra forma de sortear un falso positivo consiste en deshabilitar la regla que coincide con la entrada que el firewall de aplicaciones web ha considerado malintencionada. Como ha analizado los registros de WAF y la regla se ha restringido a 942110, puede deshabilitarla en Azure Portal. Consulte [Personalización de reglas de Firewall de aplicaciones web con Azure Portal](../ag/application-gateway-customize-waf-rules-portal.md#disable-rule-groups-and-rules).
 
Deshabilitar una regla es una ventaja cuando está seguro de que todas las solicitudes que cumplen esa condición específica son, de hecho, solicitudes legítimas, o cuando está seguro de que la regla sencillamente no se aplica a su entorno (por ejemplo, deshabilitar una regla de inyección de código SQL porque tiene back-ends que no son de SQL). 
 
Sin embargo, deshabilitar una regla es una configuración global que se aplica a todos los hosts de front-end asociados a la directiva de WAF. Al elegir deshabilitar una regla, es posible que deje las vulnerabilidades expuestas sin protección ni detección para cualquier otro host de front-end asociado a la directiva de WAF.
 
Si desea usar Azure PowerShell para deshabilitar una regla administrada, consulte la documentación del objeto [`PSAzureManagedRuleOverride`](/powershell/module/az.frontdoor/new-azfrontdoorwafmanagedruleoverrideobject). Si desea usar la CLI de Azure, consulte la documentación de [`az network front-door waf-policy managed-rules override`](/cli/azure/network/front-door/waf-policy/managed-rules/override).

![Reglas de WAF](../media/waf-front-door-tuning/waf-rules.png)

> [!TIP]
> Es conveniente documentar los cambios que realice en la directiva de WAF. Incluya solicitudes de ejemplo para ilustrar la detección de falsos positivos y explique claramente por qué ha agregado una regla personalizada, ha deshabilitado una regla o un conjunto de reglas o ha agregado una excepción. Esta documentación puede ser útil si se rediseña la aplicación en el futuro y es necesario comprobar que los cambios siguen siendo válidos. También puede ser de ayuda si alguna vez se audita o es necesario justificar por qué se ha reconfigurado la directiva de WAF desde su configuración predeterminada.

## <a name="finding-request-fields"></a>Búsqueda de campos de solicitud

Con un proxy de explorador como [Fiddler](https://www.telerik.com/fiddler), puede inspeccionar solicitudes individuales y determinar a qué campos específicos de una página web se llama. Esto es útil cuando debemos excluir determinados campos de la inspección mediante listas de exclusión en WAF.

### <a name="finding-request-attribute-names"></a>Búsqueda de nombres de atributo de solicitud
 
En este ejemplo, puede ver que el campo donde se ha escrito la cadena `1=1` se denomina `comment`. Estos datos se pasaron en el cuerpo de una solicitud POST.

![Solicitud de Fiddler que muestra el cuerpo](../media/waf-front-door-tuning/fiddler-request-attribute-name.png)

Es un campo que se puede excluir. Para más información sobre las lista de exclusión, consulte [Listas de exclusión del firewall de aplicaciones web](./waf-front-door-exclusion.md). En este caso puede excluir la evaluación mediante la configuración de la exclusión siguiente:

![Regla de exclusión](../media/waf-front-door-tuning/fiddler-request-attribute-name-exclusion.png)

También puede examinar los registros del firewall para obtener la información y ver lo que tiene que agregar a la lista de exclusión. Para habilitar el registro, consulte [Supervisión de métricas y registros en Azure Front Door](./waf-front-door-monitor.md).

Examine el registro del firewall en el archivo `PT1H.json` para la hora a la que se haya producido la solicitud que quiere inspeccionar. Los archivos `PT1H.json` están disponibles en los contenedores de cuenta de almacenamiento donde se almacenan los registros de diagnóstico `FrontDoorWebApplicationFirewallLog` y `FrontDoorAccessLog`.

En este ejemplo, puede ver la regla que bloqueó la solicitud (con la misma referencia de transacción) y se produjo exactamente al mismo tiempo:

```json
{
    "time": "2020-09-24T16:43:04.5422943Z",
    "resourceId": "/SUBSCRIPTIONS/<Subscription ID>/RESOURCEGROUPS/<Resource Group Name>/PROVIDERS/MICROSOFT.NETWORK/FRONTDOORS/AFDWAFDEMOSITE",
    "category": "FrontdoorWebApplicationFirewallLog",
    "operationName": "Microsoft.Network/FrontDoor/WebApplicationFirewallLog/Write",
    "properties": {
        "clientIP": "1.1.1.1",
        "clientPort": "53566",
        "socketIP": "1.1.1.1",
        "requestUri": "http://afdwafdemosite.azurefd.net:80/api/Feedbacks/",
        "ruleName": "DefaultRuleSet-1.0-SQLI-942110",
        "policy": "AFDWAFDemoPolicy",
        "action": "Block",
        "host": "afdwafdemosite.azurefd.net",
        "trackingReference": "0mMxsXwAAAABEalekYeI4S55qpi5R7R0/V1NURURHRTA4MTIAZGI4NGQzZDgtNWQ5Ny00ZWRkLTg2ZGYtZDJjNThlMzI2N2I4",
        "policyMode": "prevention",
        "details": {
            "matches": [
                {
                    "matchVariableName": "PostParamValue:comment",
                    "matchVariableValue": "\"1=1\""
                }
            ],
            "msg": "SQL Injection Attack: Common Injection Testing Detected",
            "data": "Matched Data: \"1=1\" found within PostParamValue:comment: \"1=1\""
        }
    }
}
```

Con su conocimiento del funcionamiento de los conjuntos de reglas administrados por Azure (consulte [Web Application Firewall en Azure Front Door](afds-overview.md)), sabe que la regla con la propiedad *action: Block* se bloquea en función de los datos coincidentes en el cuerpo de la solicitud. Puede ver en los detalles que ha coincidido con un patrón (`1=1`) y que el campo se denomina `comment`. Siga los mismos pasos anteriores para excluir el nombre de los argumentos de publicación del cuerpo de la solicitud `comment`.

### <a name="finding-request-header-names"></a>Búsqueda de nombres de encabezado de solicitud

Fiddler vuelve a ser una herramienta útil para buscar nombres de encabezado de solicitud. En la captura de pantalla siguiente puede ver los encabezados para esta solicitud GET, que incluyen Content-Type, User-Agent, etc. También puede usar encabezados de solicitud para crear exclusiones y reglas personalizadas en WAF.

![Solicitud de Fiddler que muestra el encabezado](../media/waf-front-door-tuning/fiddler-request-header-name.png)

Otra manera de ver los encabezados de solicitud y respuesta consiste en buscar dentro de las herramientas de desarrollo del explorador, como Edge o Chrome. Puede presionar F12 o hacer clic con el botón derecho -> **Inspeccionar** -> **Herramientas de desarrollo** y seleccionar la pestaña **Network** (Red). Cargue una página web y haga clic en la solicitud que quiera inspeccionar.

![Solicitud del inspector de red](../media/waf-front-door-tuning/network-inspector-request.png)

### <a name="finding-request-cookie-names"></a>Búsqueda de nombres de cookies de solicitud

Si la solicitud contiene cookies, se puede seleccionar la pestaña Cookies para verlas en Fiddler. La información de las cookies también se puede usar para crear exclusiones o reglas personalizadas en WAF.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el [firewall de aplicaciones web de Azure](../overview.md).
- Aprenda a [crear una instancia de Front Door](../../frontdoor/quickstart-create-front-door.md).
