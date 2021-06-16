---
title: Listas de exclusión y límites de tamaño de solicitud del firewall de aplicaciones web en Azure Application Gateway (Azure Portal)
description: En este artículo se proporciona información acerca de la configuración de listas de exclusión y límites de tamaño de solicitud del Firewall de aplicaciones web en Application Gateway con Azure Portal.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.date: 02/20/2020
ms.author: victorh
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e6403b1bab2bcdd6a001c5f55b34519b9447194d
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110668633"
---
# <a name="web-application-firewall-request-size-limits-and-exclusion-lists"></a>Listas de exclusión y límites de tamaño de solicitud del Firewall de aplicaciones web

El Firewall de aplicaciones web (WAF) de Azure Application Gateway proporciona protección a las aplicaciones web. En este artículo se describe la configuración de listas de exclusión y límites de tamaño de solicitud del WAF. Esta configuración se encuentra en la Directiva WAF asociada a su Application Gateway. Para más información sobre las Directivas de WAF, consulte [el Firewall de aplicaciones Web de Azure en Azure Application Gateway](ag-overview.md) y [crear directivas de Firewall de aplicaciones web para Application Gateway](create-waf-policy-ag.md)

## <a name="waf-exclusion-lists"></a>Listas de exclusión del WAF

![Límites de tamaño de solicitud](../media/application-gateway-waf-configuration/waf-policy.png)

Las listas de exclusión del WAF le permiten omitir determinados atributos de solicitud de una evaluación del WAF. Un ejemplo común son los tokens insertados de Active Directory que se usan para campos de contraseña o autenticación. Estos atributos suelen contener caracteres especiales que podrían desencadenar un falso positivo de las reglas de WAF. Una vez que se agrega un atributo a la lista de exclusión del WAF, ninguna regla de WAF configurada y activa lo toma en consideración. Las listas de exclusión tienen un ámbito global.

Los siguientes atributos se pueden agregar a las listas de exclusión por su nombre. Los valores del campo elegido no se evalúan con respecto a las reglas de WAF, pero sus nombres sí se usan (vea el ejemplo 1 a continuación, donde el encabezado User-Agent se excluye de la evaluación de WAF). Las listas de exclusión quitan la inspección del valor del campo.

* Encabezados de solicitud
* Cookies de solicitud
* Se puede agregar el nombre de atributo de la solicitud (args) como un elemento de exclusión, por ejemplo:

   * Nombre del campo de formulario
   * Entidad JSON
   * Argumentos de cadena de consulta de URL

Puede especificar una coincidencia exacta con un atributo de cadena de consulta, una cookie, el cuerpo o el encabezado de una solicitud.  Si lo desea, también puede especificar coincidencias parciales. Las reglas de exclusión tienen un ámbito global y se aplican a todas las páginas y todas las reglas.

Estos son los operadores de criterios de coincidencia admitidos:

- **Equals** (es igual a):  este operador se usa para una coincidencia exacta. Por ejemplo, para seleccionar el encabezado denominado **bearerToken**, utilice el operador de igualdad con el selector definido como **bearerToken**.
- **Starts with** (empieza por): este operador coincide con todos los campos que comienzan por el valor del selector especificado.
- **Ends with** (termina en):  este operador coincide con todos los campos de solicitud que terminan con el valor del selector especificado.
- **Contains** (contiene): este operador coincide con todos los campos de solicitud que contienen el valor del selector especificado.
- **Equals any** (es igual a cualquiera): este operador coincide con todos los campos de solicitud. * será el valor del selector.

En todos los casos, la coincidencia distingue mayúsculas de minúsculas y no se permiten expresiones regulares como selectores.

> [!NOTE]
> Para obtener más información y ayuda para solucionar problemas, vea [Solución de problemas de WAF](web-application-firewall-troubleshoot.md).

### <a name="examples"></a>Ejemplos

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

En los ejemplos siguientes se muestra el uso de las exclusiones.

#### <a name="example-1"></a>Ejemplo 1

En este ejemplo, quiere excluir el encabezado user-agent. El encabezado de solicitud user-agent contiene una cadena característica que permite a los equipos del mismo nivel del protocolo de red identificar el tipo de aplicación, el sistema operativo, el fabricante de software o la versión de software del agente de usuario de software que realiza la solicitud. Para obtener más información, consulte [User-Agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent).

Puede haber varias razones para deshabilitar la evaluación de este encabezado. Podría haber una cadena que el WAF considera malintencionada. Por ejemplo, el típico ataque de SQL "x=x" en una cadena. En algunos casos podría ser tráfico legítimo, por lo que tal vez deba excluir este encabezado de la evaluación del WAF.

El siguiente cmdlet de Azure PowerShell excluye el encabezado user-agent de la evaluación:

```azurepowershell
$exclusion1 = New-AzApplicationGatewayFirewallExclusionConfig `
   -MatchVariable "RequestHeaderNames" `
   -SelectorMatchOperator "Equals" `
   -Selector "User-Agent"
```
#### <a name="example-2"></a>Ejemplo 2

En este ejemplo se excluye el valor del parámetro *user* que se pasa en la solicitud a través de la dirección URL. Por ejemplo, supongamos que es habitual en su entorno que el campo de usuario contenga una cadena que el WAF considera contenido malintencionado y, por tanto, la bloquea.  En este caso, puede excluir el parámetro user para que el WAF no evalúe nada en el campo.

El siguiente cmdlet de Azure PowerShell excluye el parámetro user de la evaluación:

```azurepowershell
$exclusion2 = New-AzApplicationGatewayFirewallExclusionConfig `
   -MatchVariable "RequestArgNames" `
   -SelectorMatchOperator "StartsWith" `
   -Selector "user"
```
Por tanto, si la dirección URL `http://www.contoso.com/?user%281%29=fdafdasfda` se pasa a WAF, no evaluará la cadena **fdafdasfda**, pero sí el nombre de parámetro **user%281%29**. 

## <a name="waf-request-size-limits"></a>Límites de tamaño de solicitud del WAF



El firewall de aplicaciones web le permite configurar límites de tamaño de solicitud dentro de los límites inferior y superior. Hay disponibles dos configuraciones de límites de tamaño:

- El campo de tamaño máximo del cuerpo de la solicitud se especifica en kilobytes y controla el límite de tamaño de la solicitud general, excluida cualquier carga de archivo. Este campo tiene un valor mínimo de 1 KB y un valor máximo de 128 KB. El valor predeterminado para el tamaño del cuerpo de la solicitud es de 128 KB.
- El campo de límite de carga de archivo se especifica en MB y controla el tamaño máximo de carga de archivo permitido. Este campo puede tener un valor mínimo de 1 MB y los siguientes máximos:

   - 100 MB para puertas de enlaces v1 medianas WAF
   - 500 MB para puertas de enlace v1 grandes WAF
   - 750 MB para puertas de enlace WAF, versión 2 

 El valor predeterminado del límite de carga de archivo es 100 MB.

El WAF también ofrece un botón configurable para activar o desactivar la inspección del cuerpo de la solicitud. De forma predeterminada, la inspección del cuerpo de la solicitud está habilitada. Si se desactiva la inspección del cuerpo de la solicitud, el WAF no evalúa el contenido del cuerpo del mensaje HTTP. En tal caso, el WAF continúa aplicando reglas de WAF en URI, cookies y encabezados. Si se desactiva la inspección del cuerpo de la solicitud, el campo de tamaño máximo del cuerpo de la solicitud no es aplicable y no se puede definir. Al desactivar la inspección del cuerpo de la solicitud, se podrán enviar mensajes de más de 128 KB al WAF, pero no se inspecciona el cuerpo de mensaje en busca de vulnerabilidades.

## <a name="next-steps"></a>Pasos siguientes

Después de configurar el WAF, puede aprender a ver los registros del WAF. Para más información, consulte [Diagnósticos de Application Gateway](../../application-gateway/application-gateway-diagnostics.md#diagnostic-logging).
