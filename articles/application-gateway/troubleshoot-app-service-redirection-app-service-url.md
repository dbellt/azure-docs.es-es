---
title: Solución de problemas de redireccionamiento a la dirección URL de App Service
titleSuffix: Azure Application Gateway
description: En este artículo se proporciona información sobre cómo solucionar el problema de redireccionamiento cuando se utiliza Azure Application Gateway con Azure App Service
services: application-gateway
author: jaesoni
ms.service: application-gateway
ms.topic: troubleshooting
ms.date: 04/15/2021
ms.author: jaysoni
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d2291bc88a90a703239764a2d5fda9b2889a7af7
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108319672"
---
# <a name="troubleshoot-app-service-issues-in-application-gateway"></a>Solución de problemas de App Service en Application Gateway

Obtenga información acerca de cómo diagnosticar y resolver los problemas que pueden surgir cuando se usa Azure App Service como destino de back-end con Azure Application Gateway.

## <a name="overview"></a>Información general

En este artículo aprenderá a resolver los siguientes problemas:

* La dirección URL de App Service se expone en el explorador cuando hay un redireccionamiento.
* Dominio de la cookie ARRAffinity de App Service establecido en el nombre de host de App Service (example.azurewebsites.net) en lugar del host original.

Cuando una aplicación back-end envía una respuesta de redireccionamiento, es posible que le interese redirigir el cliente a una dirección URL diferente de la especificada por la aplicación back-end. Es posible que quiera hacerlo cuando un servicio de aplicaciones se hospeda detrás de una puerta de enlace de aplicaciones y requiere que el cliente realice un redireccionamiento a su ruta de acceso relativa. Un ejemplo es un redireccionamiento de contoso.azurewebsites.net/path1 a contoso.azurewebsites.net/path2. 

Cuando el servicio de aplicaciones envía una respuesta de redireccionamiento, usa el mismo nombre de host en el encabezado de ubicación de su respuesta que el que aparece en la solicitud que recibe de la puerta de enlace de aplicaciones. Por ejemplo, el cliente hace la solicitud directamente a contoso.azurewebsites.net/path2, en lugar de pasar por la puerta de enlace de aplicaciones contoso.com/path2. No quiere omitir la puerta de enlace de aplicaciones.

Este problema puede ocurrir debido a los siguientes motivos principales:

- Ha configurado el redireccionamiento en su instancia de App Service. El redireccionamiento puede ser tan sencillo como agregar una barra diagonal final a la solicitud.
- Tiene autenticación de Azure Active Directory, lo que provoca el redireccionamiento.

Además, cuando se usa App Service detrás de una puerta de enlace de aplicaciones, el nombre de dominio asociado a la puerta de enlace de aplicaciones (example.com) es diferente del nombre de dominio de App Service (por ejemplo, example.azurewebsites.net). El valor de dominio para la cookie ARRAffinity establecida por App Service incluye el nombre de dominio example.azurewebsites.net, lo cual no es deseable. El nombre de host original (example.com) debe ser el valor del nombre de dominio de la cookie.

## <a name="sample-configuration"></a>Configuración de ejemplo

- Cliente de escucha HTTP: básico o de varios sitios
- Grupo de direcciones de back-end: App Service
- Configuración HTTP: opción **Pick Hostname from Backend Address** (Seleccionar el nombre de host de la dirección de back-end) habilitada
- Sondeo: opción **Pick Hostname from HTTP Settings** (Seleccionar el nombre de host de la configuración HTTP) habilitada

## <a name="cause"></a>Causa

App Service es un servicio multiinquilino, por lo que usa el encabezado de host en la solicitud para enrutar la solicitud al punto de conexión correcto. El nombre de dominio predeterminado de App Service, *.azurewebsites.net (por ejemplo, contoso.azurewebsites.net), es diferente del nombre de dominio de la puerta de enlace de aplicaciones (por ejemplo, contoso.com). 

La solicitud original del cliente tiene el nombre de dominio de la puerta de enlace de aplicaciones (contoso.com) como nombre de host. Debe configurar la puerta de enlace de aplicaciones para cambiar el nombre de host de la solicitud original al nombre de host de App Service cuando enruta la solicitud al back-end de App Service. Use el conmutador **Seleccionar el nombre de host de la dirección de back-end** en la configuración HTTP de la puerta de enlace de aplicaciones. Use el conmutador **Seleccionar el nombre de host de la configuración de HTTP de back-end** en la configuración del sondeo de mantenimiento.



![La puerta de enlace de aplicaciones cambia el nombre de host.](./media/troubleshoot-app-service-redirection-app-service-url/appservice-1.png)

Cuando App Service realiza un redireccionamiento, usa el nombre de host reemplazado contoso.azurewebsites.net en el encabezado de ubicación, en lugar del nombre de host original contoso.com, a menos que esté configurado de otro modo. Compruebe los encabezados de solicitud y respuesta de ejemplo siguientes.
```
## Request headers to Application Gateway:

Request URL: http://www.contoso.com/path

Request Method: GET

Host: www.contoso.com

## Response headers:

Status Code: 301 Moved Permanently

Location: http://contoso.azurewebsites.net/path/

Server: Microsoft-IIS/10.0

Set-Cookie: ARRAffinity=b5b1b14066f35b3e4533a1974cacfbbd969bf1960b6518aa2c2e2619700e4010;Path=/;HttpOnly;Domain=contoso.azurewebsites.net

X-Powered-By: ASP.NET
```
En el ejemplo anterior, observe que el encabezado de respuesta tiene un código de estado 301 para el redireccionamiento. El encabezado de ubicación tiene el nombre de host de la instancia de App Service en lugar del nombre de host original, `www.contoso.com`.

## <a name="solution-rewrite-the-location-header"></a>Solución: reescribir el encabezado de ubicación

Establezca el nombre de host del encabezado de ubicación en el nombre de dominio de la puerta de enlace de aplicaciones. Para ello, cree una [regla de reescritura](./rewrite-http-headers-url.md) con una condición que evalúe si el encabezado de ubicación en la respuesta contiene azurewebsites.net. También debe realizar una acción para volver a escribir el encabezado de ubicación de modo que tenga el nombre de host de la puerta de enlace de aplicaciones. Para obtener más información, vea las instrucciones para [volver a escribir el encabezado de ubicación](./rewrite-http-headers-url.md#modify-a-redirection-url).

> [!NOTE]
> La compatibilidad con la reescritura de encabezados HTTP solo está disponible para [Standard_v2 y WAF_v2 SKU](./application-gateway-autoscaling-zone-redundant.md) de Application Gateway. Se recomienda [migrar a v2](./migrate-v1-v2.md) para la reescritura de encabezados y otras [funcionalidades avanzadas](./application-gateway-autoscaling-zone-redundant.md#feature-comparison-between-v1-sku-and-v2-sku) que están disponibles con la versión v2 de la SKU.

## <a name="alternate-solution-use-a-custom-domain-name"></a>Solución alternativa: utilizar un nombre de dominio personalizado

La característica de dominio personalizado de App Service es otra solución para redirigir siempre el tráfico al nombre de dominio de Application Gateway (en este ejemplo, `www.contoso.com`). Esta configuración también sirve como solución para el problema de las cookies de ARRAffinity. De forma predeterminada, el dominio de cookies de ARRAffinity se establece en el nombre de host predeterminado de App Service (example.azurewebsites.net) en lugar del nombre de dominio de Application Gateway. Por lo tanto, en casos como este, el explorador rechazará la cookie debido a la diferencia en los nombres de dominio de la solicitud y la cookie.

Puede seguir el método especificado para los problemas de discrepancias del dominio de cookies de redireccionamiento y ARRAffinity. Este método necesitará tener acceso a la zona DNS del dominio personalizado.

**Paso 1**: establezca un dominio personalizado en App Service y compruebe la propiedad del dominio agregando los [registros DNS CNAME y TXT](../app-service/app-service-web-tutorial-custom-domain.md#get-a-domain-verification-id).
Los registros tendrán un aspecto similar a:
-  `www.contoso.com` IN CNAME `contoso.azurewebsite.net`
-  `asuid.www.contoso.com` IN TXT "`<verification id string>`"


**Paso 2**: el registro CNAME del paso anterior solo era necesario para la comprobación del dominio. En última instancia, necesitamos que el tráfico se enrute a través de Application Gateway. Por lo tanto, ahora puede modificar el registro CNAME de `www.contoso.com` para que apunte al nombre de dominio completo de Application Gateway. Para establecer un nombre de dominio completo para Application Gateway, vaya a su recurso de dirección IP pública y asígnele una etiqueta de nombre DNS. El registro CNAME actualizado debería tener un aspecto similar a: 
-  `www.contoso.com` IN CNAME `contoso.eastus.cloudapp.azure.com`


**Paso 3**: deshabilite "Seleccionar el nombre de host de la dirección de back-end" para la configuración HTTP asociada.

En PowerShell, no use el conmutador `-PickHostNameFromBackendAddress` en el comando `Set-AzApplicationGatewayBackendHttpSettings`.


**Paso 4**: para que los sondeos determinen que el estado del back-end es correcto y que el tráfico es operativo, establezca un sondeo de estado personalizado con el campo de host como dominio personalizado o predeterminado de la instancia de App Service.

En PowerShell, no use el conmutador `-PickHostNameFromBackendHttpSettings` en el comando `Set-AzApplicationGatewayProbeConfig` y utilice el dominio personalizado o predeterminado de la instancia de App Service en el conmutador -HostName del sondeo.

Para implementar los pasos mencionados anteriormente con PowerShell en una instalación existente, use el siguiente script de Powershell de ejemplo. Tenga en cuenta que no hemos usado los modificadores **-PickHostname** en la configuración de sondeo y configuración de HTTP.

```azurepowershell-interactive
$gw=Get-AzApplicationGateway -Name AppGw1 -ResourceGroupName AppGwRG
Set-AzApplicationGatewayProbeConfig -ApplicationGateway $gw -Name AppServiceProbe -Protocol Http -HostName "example.azurewebsites.net" -Path "/" -Interval 30 -Timeout 30 -UnhealthyThreshold 3
$probe=Get-AzApplicationGatewayProbeConfig -Name AppServiceProbe -ApplicationGateway $gw
Set-AzApplicationGatewayBackendHttpSettings -Name appgwhttpsettings -ApplicationGateway $gw -Port 80 -Protocol Http -CookieBasedAffinity Disabled -Probe $probe -RequestTimeout 30
Set-AzApplicationGateway -ApplicationGateway $gw
```
  ```
  ## Request headers to Application Gateway:

  Request URL: http://www.contoso.com/path

  Request Method: GET

  Host: www.contoso.com

  ## Response headers:

  Status Code: 301 Moved Permanently

  Location: http://www.contoso.com/path/

  Server: Microsoft-IIS/10.0

  Set-Cookie: ARRAffinity=b5b1b14066f35b3e4533a1974cacfbbd969bf1960b6518aa2c2e2619700e4010;Path=/;HttpOnly;Domain=www.contoso.com

  X-Powered-By: ASP.NET
  ```
  ## <a name="next-steps"></a>Pasos siguientes

Si los pasos anteriores no resuelven el problema, abra una [incidencia de soporte técnico](https://azure.microsoft.com/support/options/).
