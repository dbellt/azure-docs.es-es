---
title: Solución de errores de puerta de enlace incorrecta en Azure Application Gateway
description: 'Obtenga información sobre cómo solucionar errores de Application Gateway: 502, el servidor web ha recibido una respuesta no válida mientras actuaba como puerta de enlace o servidor proxy.'
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: troubleshooting
ms.date: 11/16/2019
ms.author: amsriva
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 2f1892ab7cafae0b344026ed73a801ca30f4d9ee
ms.sourcegitcommit: ef950cf37f65ea7a0f583e246cfbf13f1913eb12
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111421562"
---
# <a name="troubleshooting-bad-gateway-errors-in-application-gateway"></a>Solución de errores de puerta de enlace incorrecta en el servicio Puerta de enlace de aplicaciones

Aprenda a solucionar problemas de errores de puerta de enlace incorrecta (502) recibidos al usar Azure Application Gateway.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="overview"></a>Información general

Después de configurar una instancia de Application Gateway, uno de los errores que pueden encontrar los usuarios es "Error del servidor: 502 - Web server received an invalid response while acting as a gateway or proxy server". (502: el servidor web ha recibido una respuesta no válida mientras actuaba como puerta de enlace o servidor proxy". Este error puede ocurrir debido a los siguientes motivos principales:

* Un grupo de seguridad de red, una ruta definida por el usuario o un DNS personalizado están bloqueando el acceso a los miembros del grupo de back-end.
* Las VM de back-end o instancias del conjunto de escalado de máquinas virtuales no responden al sondeo de estado predeterminado.
* La configuración de las sondas de estado personalizadas no es válida o adecuada.
* [El grupo back-end de la instancia de Azure Application Gateway no está configurado o está vacío](#empty-backendaddresspool).
* Ninguna de las máquinas virtuales o instancias del [conjunto de escalado de máquina virtual está en buen estado](#unhealthy-instances-in-backendaddresspool).
* [El tiempo de espera de solicitud se superó o hay problemas de conectividad](#request-time-out) con las solicitudes de usuario.

## <a name="network-security-group-user-defined-route-or-custom-dns-issue"></a>Problema con grupo de seguridad de red, ruta definida por el usuario o DNS personalizado

### <a name="cause"></a>Causa

Si el acceso al back-end se bloquea debido a un grupo de seguridad de red, una ruta definida por el usuario o un DNS personalizado, las instancias de Application Gateway no podrán alcanzar el grupo de back-end. Esto provoca fallos de sondeo y genera errores 502.

Tenga en cuenta que el grupo de seguridad de red y la ruta definida por el usuario pueden estar presentes en la subred de Application Gateway o en la subred donde están implementadas las VM de la aplicación.

De forma similar, la presencia de un DNS personalizado en la red virtual también puede provocar problemas. Es posible que un FQDN utilizado para los miembros del grupo de back-end no se resuelva correctamente en el servidor DNS configurado por el usuario para la red virtual.

### <a name="solution"></a>Solución

Valide la configuración de NSG, UDR y DNS mediante los pasos siguientes:

* Compruebe los grupos de seguridad de red asociados a la subred de Application Gateway. Asegúrese de que la comunicación con el back-end no se bloquee.
* Compruebe la ruta definida por el usuario asociada a la subred de Application Gateway. Asegúrese de que la ruta definida por el usuario no esté alejando el tráfico de la subred de back-end. Por ejemplo, compruebe el enrutamiento a aplicaciones virtuales de red o las rutas predeterminadas anunciadas en la subred de Application Gateway mediante ExpressRoute/VPN.

```azurepowershell
$vnet = Get-AzVirtualNetwork -Name vnetName -ResourceGroupName rgName
Get-AzVirtualNetworkSubnetConfig -Name appGwSubnet -VirtualNetwork $vnet
```

* Compruebe el grupo de seguridad de red y la ruta efectivos con la máquina virtual de back-end.

```azurepowershell
Get-AzEffectiveNetworkSecurityGroup -NetworkInterfaceName nic1 -ResourceGroupName testrg
Get-AzEffectiveRouteTable -NetworkInterfaceName nic1 -ResourceGroupName testrg
```

* Compruebe la presencia de DNS personalizado en la red virtual. El DNS se puede comprobar examinando los detalles de las propiedades de red virtual en la salida.

```json
Get-AzVirtualNetwork -Name vnetName -ResourceGroupName rgName 
DhcpOptions            : {
                           "DnsServers": [
                             "x.x.x.x"
                           ]
                         }
```
Si está presente, asegúrese de que el servidor DNS pueda resolver correctamente el FQDN de los miembros del grupo de back-end.

## <a name="problems-with-default-health-probe"></a>Problemas con la sonda de estado predeterminada

### <a name="cause"></a>Causa

Los errores 502 también pueden ser indicativos frecuentes de que el sondeo de estado predeterminada no puede conectarse con las VM de back-end.

Cuando se aprovisiona una instancia de Application Gateway, se configura automáticamente un sondeo de estado predeterminado para cada elemento BackendAddressPool que utiliza las propiedades de BackendHttpSetting. El usuario no tiene que escribir datos para establecer esta sonda. En concreto, cuando se configura una regla de equilibrio de carga, se crea una asociación entre un elemento BackendHttpSetting y otro BackendAddressPool. Se ha configurado un sondeo predeterminado para cada una de estas asociaciones y la instancia de Application Gateway inicia una conexión de comprobación de estado periódica en cada instancia del elemento BackendAddressPool en el puerto especificado en el elemento BackendHttpSetting. 

En la tabla siguiente se enumeran los valores asociados con el sondeo de estado predeterminado.

| Propiedad de sondeo | Value | Descripción |
| --- | --- | --- |
| Dirección URL de sondeo |`http://127.0.0.1/` |Ruta de acceso URL |
| Intervalo |30 |Intervalo de sondeo en segundos |
| Tiempo de espera |30 |Tiempo de espera del sondeo en segundos |
| Umbral incorrecto |3 |Número de reintentos de sondeo. El servidor back-end se marca como inactivo después de que el número de errores de sondeo consecutivos alcanza el umbral incorrecto. |

### <a name="solution"></a>Solución

* Asegúrese de que se ha configurado un sitio predeterminado y de que está escuchando en 127.0.0.1.
* Si BackendHttpSetting especifica un puerto distinto de 80, se debe configurar que el sitio predeterminado escuche en ese puerto.
* La llamada a `http://127.0.0.1:port` debe devolver el código de resultado HTTP 200. Debe devolverse dentro del periodo de espera de 30 segundos.
* Asegúrese de que el puerto configurado está abierto y de que no hay ninguna regla de firewall ni grupos de seguridad de red de Azure que bloqueen el tráfico entrante o saliente en dicho puerto.
* Si usa el servicio en la nube o las VM clásicas de Azure con un FQDN o una IP pública, asegúrese de que esté abierto el [punto de conexión](/previous-versions/azure/virtual-machines/windows/classic/setup-endpoints?toc=%2fazure%2fapplication-gateway%2ftoc.json) correspondiente.
* Si la VM se configura mediante Azure Resource Manager y se encuentra fuera de la red virtual donde se implementó la instancia de Application Gateway, se debe configurar un [grupo de seguridad de red](../virtual-network/network-security-groups-overview.md) para permitir el acceso en el puerto deseado.

## <a name="problems-with-custom-health-probe"></a>Problemas con la sonda de estado personalizada

### <a name="cause"></a>Causa

Las sondas de estado personalizadas dotan al comportamiento de sondeo predeterminado de mayor flexibilidad. Si usa sondeos personalizados, puede configurar el intervalo de sondeo, la dirección URL, la ruta de acceso a la prueba y la cantidad de respuestas erróneas que se aceptan antes de marcar la instancia del grupo de back-end como incorrecta.

Se han agregado las siguientes propiedades adicionales:

| Propiedad de sondeo | Descripción |
| --- | --- |
| Nombre |Nombre del sondeo. Este nombre se usa para hacer referencia al sondeo en la configuración de HTTP de back-end. |
| Protocolo |Protocolo usado para enviar el sondeo. El sondeo utiliza el protocolo definido en la configuración de HTTP del back-end. |
| Host |Nombre de host para enviar el sondeo. Solo es aplicable cuando se ha configurado un entorno multisitio en la instancia de Application Gateway. Es diferente al nombre de host de máquina virtual. |
| Path |Ruta de acceso relativa del sondeo. La ruta de acceso válida se inicia desde '/'. El sondeo se envía a \<protocol\>://\<host\>:\<port\>\<path\> |
| Intervalo |Intervalo de sondeo en segundos. Es el intervalo de tiempo entre dos sondeos consecutivos. |
| Tiempo de espera |Tiempo de espera del sondeo en segundos. Si no se recibe una respuesta válida dentro del período de espera, el sondeo se marca como erróneo. |
| Umbral incorrecto |Número de reintentos de sondeo. El servidor back-end se marca como inactivo después de que el número de errores de sondeo consecutivos alcanza el umbral incorrecto. |

### <a name="solution"></a>Solución

Valide que el sondeo de mantenimiento personalizado esté configurado correctamente según la tabla anterior. Además de realizar los pasos de solución de problemas anteriores, no se olvide tampoco de llevar a cabo estos:

* Asegúrese de que la sonda se ha especificado correctamente según la [guía](application-gateway-create-probe-ps.md).
* Si la instancia de Application Gateway está configurada para un único sitio, el nombre de host debería especificarse de forma predeterminada como `127.0.0.1`, salvo que se configure de otra manera en el sondeo personalizado.
* Asegúrese de que una llamada a http://\<host\>:\<port\>\<path\> devuelve el código de resultado HTTP 200.
* Compruebe que los valores de Interval, Timeout y UnhealtyThreshold estén dentro de los rangos aceptables.
* Si utiliza un sondeo HTTPS, asegúrese de que el servidor de back-end no requiera SNI; para ello, configure un certificado de reserva en dicho servidor.

## <a name="request-time-out"></a>Tiempo de espera de solicitud superado

### <a name="cause"></a>Causa

Cuando se recibe una solicitud de usuario, la instancia de Application Gateway aplica las reglas configuradas a la solicitud y la enruta a una instancia del grupo de back-end. Espera, durante un intervalo de tiempo que puede configurarse, una respuesta de la instancia back-end. De forma predeterminada, este intervalo es de **20 segundos**. En Application Gateway v1, si la instancia de Application Gateway no recibe una respuesta de la aplicación de back-end en este intervalo, la solicitud de usuario obtiene un error 502.  En Application Gateway v2, si la instancia de Application Gateway no recibe una respuesta de la aplicación de back-end en este intervalo, la solicitud se probará en un segundo miembro del grupo de back-end.  Si se produce un error en la segunda solicitud, la solicitud de usuario obtiene un error 502.

### <a name="solution"></a>Solución

La instancia de Application Gateway le permite configurar esta opción mediante BackendHttpSetting que, posteriormente, puede aplicarse a diferentes grupos. Diferentes grupos de back-end pueden tener distintos elementos BackendHttpSetting y un tiempo de espera de solicitud diferente configurado.

```azurepowershell
    New-AzApplicationGatewayBackendHttpSettings -Name 'Setting01' -Port 80 -Protocol Http -CookieBasedAffinity Enabled -RequestTimeout 60
```

## <a name="empty-backendaddresspool"></a>Valor de BackendAddressPool vacío

### <a name="cause"></a>Causa

Si la instancia de Application Gateway no tiene ninguna VM ni ningún conjunto de escalado de máquinas virtuales configurado en el grupo de direcciones de back-end, no puede enrutar ninguna solicitud de cliente y genera un error de puerta de enlace incorrecta.

### <a name="solution"></a>Solución

Asegúrese de que el grupo de direcciones de back-end no está vacío. Puede hacerlo mediante PowerShell, la CLI o el Portal.

```azurepowershell
Get-AzApplicationGateway -Name "SampleGateway" -ResourceGroupName "ExampleResourceGroup"
```

El resultado del cmdlet anterior debe contener un grupo de direcciones back-end que no esté vacío. El ejemplo siguiente muestra dos grupos devueltos que están configurados con un FQDN o una dirección IP para las VM de back-end. El estado de aprovisionamiento de BackendAddressPool debe ser "Correcto".

BackendAddressPoolsText:

```json
[{
    "BackendAddresses": [{
        "ipAddress": "10.0.0.10",
        "ipAddress": "10.0.0.11"
    }],
    "BackendIpConfigurations": [],
    "ProvisioningState": "Succeeded",
    "Name": "Pool01",
    "Etag": "W/\"00000000-0000-0000-0000-000000000000\"",
    "Id": "/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Network/applicationGateways/<application gateway name>/backendAddressPools/pool01"
}, {
    "BackendAddresses": [{
        "Fqdn": "xyx.cloudapp.net",
        "Fqdn": "abc.cloudapp.net"
    }],
    "BackendIpConfigurations": [],
    "ProvisioningState": "Succeeded",
    "Name": "Pool02",
    "Etag": "W/\"00000000-0000-0000-0000-000000000000\"",
    "Id": "/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Network/applicationGateways/<application gateway name>/backendAddressPools/pool02"
}]
```

## <a name="unhealthy-instances-in-backendaddresspool"></a>Instancias incorrectas en BackendAddressPool

### <a name="cause"></a>Causa

Si todas las instancias de BackendAddressPool son incorrectas, la instancia de Application Gateway no tendrá ningún back-end al que enrutar la solicitud de usuario. Esto también podría suceder si las instancias back-end son correctas, pero no tienen implementada la aplicación necesaria.

### <a name="solution"></a>Solución

Asegúrese de que las instancias son correctas y de que la aplicación está configurada de la forma adecuada. Compruebe si las instancias de back-end pueden responder a un ping de otra VM de la misma red virtual. Si se configura con un punto de conexión público, asegúrese de que se pueda utilizar una solicitud de explorador a la aplicación web.

## <a name="next-steps"></a>Pasos siguientes

Si los pasos anteriores no resuelven el problema, abra una [incidencia de soporte técnico](https://azure.microsoft.com/support/options/).
