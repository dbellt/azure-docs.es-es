---
title: Solución de problemas de Azure VPN Gateway mediante registros de diagnóstico
description: Solución de problemas de Azure VPN Gateway mediante registros de diagnóstico
services: vpn-gateway
author: stegag
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 03/15/2021
ms.author: stegag
ms.openlocfilehash: 96a43e9a71d5fe6cdec772ec5e32d5dd639ab068
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108134398"
---
# <a name="troubleshoot-azure-vpn-gateway-using-diagnostic-logs"></a>Solución de problemas de Azure VPN Gateway mediante registros de diagnóstico

En este artículo se explican los distintos registros disponibles para el diagnóstico de VPN Gateway y cómo usarlos para solucionar problemas de VPN Gateway de forma eficaz.

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

Los registros siguientes están disponibles en Azure:

|***Nombre** _ | _ *_Descripción_** |
|---        | ---               |
|**GatewayDiagnosticLog** | Contiene los registros de diagnóstico para los eventos de configuración de puerta de enlace, los cambios principales y los eventos de mantenimiento |
|**TunnelDiagnosticLog** | Contiene los eventos de cambio del estado del túnel. Los eventos de conexión o desconexión del túnel tienen un motivo resumido para el cambio de estado si es aplicable. |
|**RouteDiagnosticLog** | Registra los cambios en rutas estáticas y eventos BGP que se producen en la puerta de enlace |
|**IKEDiagnosticLog** | Registra los mensajes de control de IKE y los eventos en la puerta de enlace |
|**P2SDiagnosticLog** | Registra mensajes de control de punto a sitio y eventos en la puerta de enlace. |

Observe que hay varias columnas disponibles en estas tablas. En este artículo, solo presentamos los más relevantes para un consumo de registro más sencillo.

## <a name="set-up-logging"></a><a name="setup"></a>Configuración de los registros

Para obtener información sobre cómo configurar eventos de registro de diagnóstico desde Azure VPN Gateway con Azure Log Analytics, consulte [Configuración de alertas de eventos de registro de recursos de VPN Gateway](./vpn-gateway-howto-setup-alerts-virtual-network-gateway-log.md).

## <a name="gatewaydiagnosticlog"></a><a name="GatewayDiagnosticLog"></a>GatewayDiagnosticLog

Los cambios de configuración se auditan en la tabla **GatewayDiagnosticLog**. Podría tardar unos minutos antes de que los cambios que se ejecuten se reflejen en los registros.

Aquí tiene una consulta de ejemplo como referencia.

```
AzureDiagnostics  
| where Category == "GatewayDiagnosticLog"  
| project TimeGenerated, OperationName, Message, Resource, ResourceGroup  
| sort by TimeGenerated asc
```

Esta consulta en **GatewayDiagnosticLog** mostrará varias columnas.

|***Nombre** _ | _ *_Descripción_** |
|---        | ---               |
|**TimeGenerated** | La marca de tiempo de cada evento, en la zona horaria UTC.|
|**OperationName** |El evento que se ha producido. Puede ser *SetGatewayConfiguration, SetConnectionConfiguration, HostMaintenanceEvent, GatewayTenantPrimaryChanged, MigrateCustomerSubscription, GatewayResourceMove, ValidateGatewayConfiguration*.|
|**Mensaje** | El detalle de la operación que está ocurriendo y muestra los resultados correctos y erróneos.|

En el ejemplo siguiente se muestra la actividad que se registra cuando se aplica una nueva configuración:

:::image type="content" source="./media/troubleshoot-vpn-with-azure-diagnostics/image-26-set-gateway.png" alt-text="Ejemplo de una operación de establecimiento de la puerta de enlace que se ha encontrado en GatewayDiagnosticLog.":::


Tenga en cuenta que se registrará una operación SetGatewayConfiguration cada vez que se modifique una configuración en un VPN Gateway o en una puerta de enlace de red local.
La referencia cruzada de los resultados de la tabla **GatewayDiagnosticLog** con las de la tabla **TunnelDiagnosticLog** puede ayudarnos a determinar si un error de conectividad de túnel se ha iniciado al mismo tiempo que se ha cambiado una configuración o se ha realizado un mantenimiento. Si es así, tenemos un excelente puntero hacia la posible causa principal.

## <a name="tunneldiagnosticlog"></a><a name="TunnelDiagnosticLog"></a>TunnelDiagnosticLog

La tabla **TunnelDiagnosticLog** es muy útil para inspeccionar los estados de conectividad históricos del túnel.

Aquí tiene una consulta de ejemplo como referencia.

```
AzureDiagnostics
| where Category == "TunnelDiagnosticLog"
| project TimeGenerated, OperationName, instance_s, Resource, ResourceGroup
| sort by TimeGenerated asc
```

Esta consulta sobre **TunnelDiagnosticLog** mostrará varias columnas.


|***Nombre** _ | _ *_Descripción_** |
|---        | ---               |
|**TimeGenerated** | La marca de tiempo de cada evento, en la zona horaria UTC.|
|**OperationName** | El evento que se ha producido. Puede ser *TunnelConnected* o *TunnelDisconnected*.|
| **Instancias\_** | La instancia de rol de puerta de enlace que desencadenó el evento. Puede ser GatewayTenantWorker\_IN\_0 o GatewayTenantWorker\_IN\_1, que son los nombres de las dos instancias de la puerta de enlace.|
| **Recurso** | Indica el nombre de la puerta de enlace de VPN. |
| **ResourceGroup** | Indica el grupo de recursos donde se encuentra la puerta de enlace.|


Salida de ejemplo:

:::image type="content" source="./media/troubleshoot-vpn-with-azure-diagnostics/image-16-tunnel-connected.png" alt-text="Ejemplo de evento conectado a un túnel detectado en TunnelDiagnosticLog":::


**TunnelDiagnosticLog** es muy útil para solucionar problemas de eventos pasados sobre desconexiones de VPN inesperadas. Su naturaleza ligera ofrece la posibilidad de analizar intervalos de tiempo grandes durante varios días con poco esfuerzo.
Solo después de identificar la marca de tiempo de una desconexión, puede cambiar al análisis más detallado de la tabla **IKEdiagnosticLog** para profundizar más en el razonamiento de las desconexiones si estas están relacionados con IPSec.


Algunas sugerencias para la solución de problemas:
- Si ve un evento de desconexión en una instancia de puerta de enlace, seguido de un evento de conexión en la instancia de puerta de enlace **diferente** en unos segundos, está examinando una conmutación por error de puerta de enlace. Normalmente, se trata de un comportamiento esperado debido al mantenimiento en una instancia de puerta de enlace. Para más información sobre este comportamiento, consulte [acerca de la redundancia de Azure VPN Gateway](./vpn-gateway-highlyavailable.md#about-azure-vpn-gateway-redundancy).
- Se observará el mismo comportamiento si ejecuta intencionadamente un restablecimiento de puerta de enlace en el lado de Azure, lo que provoca un reinicio de la instancia de puerta de enlace activa. Para más información sobre este comportamiento, consulte [Restablecimiento de una puerta de enlace o conexión de VPN](./reset-gateway.md).
- Si ve un evento de desconexión en una instancia de puerta de enlace, seguido de un evento de conexión en la **misma** instancia de puerta de enlace en pocos segundos, es posible que esté examinando un problema de red que provoca un tiempo de espera de DPD o una desconexión enviada erróneamente por el dispositivo local.

## <a name="routediagnosticlog"></a><a name="RouteDiagnosticLog"></a>RouteDiagnosticLog

La tabla **RouteDiagnosticLog** realiza un seguimiento de la actividad de rutas modificadas estáticamente o rutas recibidas mediante BGP.

Aquí tiene una consulta de ejemplo como referencia.

```
AzureDiagnostics
| where Category == "RouteDiagnosticLog"
| project TimeGenerated, OperationName, Message, Resource, ResourceGroup
```

Esta consulta en **RouteDiagnosticLog** mostrará varias columnas.

|***Nombre** _ | _ *_Descripción_** |
|---        | ---               |
|**TimeGenerated** | La marca de tiempo de cada evento, en la zona horaria UTC.|
|**OperationName** | El evento que se ha producido. Puede ser *StaticRouteUpdate, BgpRouteUpdate, BgpConnectedEvent, BgpDisconnectedEvent*.|
| **Mensaje** | el detalle de la operación que está ocurriendo.|

La salida mostrará información útil sobre los pares BGP conectados o desconectados y las rutas intercambiadas.

Ejemplo:


:::image type="content" source="./media/troubleshoot-vpn-with-azure-diagnostics/image-31-bgp-route.png" alt-text="Ejemplo de actividad de intercambio de rutas BGP que se ha detectado en RouteDiagnosticLog.":::


## <a name="ikediagnosticlog"></a><a name="IKEDiagnosticLog"></a>IKEDiagnosticLog

La tabla **IKEDiagnosticLog** ofrece un registro de depuración detallado para IKE/IPSec. Esto resulta muy útil para la revisión al solucionar problemas de escenarios de desconexiones o de errores de conexión de VPN.

Aquí tiene una consulta de ejemplo como referencia.

```
AzureDiagnostics  
| where Category == "IKEDiagnosticLog" 
| extend Message1=Message
| parse Message with * "Remote " RemoteIP ":" * "500: Local " LocalIP ":" * "500: " Message2
| extend Event = iif(Message has "SESSION_ID",Message2,Message1)
| project TimeGenerated, RemoteIP, LocalIP, Event, Level 
| sort by TimeGenerated asc
```

Esta consulta en **IKEDiagnosticLog** mostrará varias columnas.


|***Nombre** _ | _ *_Descripción_** |
|---        | ---               |
|**TimeGenerated** | La marca de tiempo de cada evento, en la zona horaria UTC.|
| **RemoteIP** | La dirección IP del dispositivo VPN local. En escenarios reales, resulta útil filtrar por la dirección IP del dispositivo local pertinente si hay más de uno. |
|**LocalIP** | La dirección IP de la VPN Gateway estamos solucionando problemas. En escenarios reales, resulta útil filtrar por la dirección IP de la puerta de enlace de VPN pertinente. Habrá más de una en la suscripción. |
|**Evento** | Contiene un mensaje de diagnóstico útil para la solución de problemas. Normalmente comienzan con una palabra clave y hacen referencia a las acciones realizadas por la puerta de enlace de Azure: **\[SEND\]** indica un evento provocado por un paquete IPSec enviado por la puerta de enlace de Azure.  **\[RECEIVED\]** indica un evento en consecuencia de un paquete recibido desde un dispositivo local.  **\[LOCAL\]** indica una acción realizada localmente por la puerta de enlace de Azure. |


Observe cómo RemoteIP, LocalIP y las columnas de eventos no están presentes en la lista de columnas original en la base de datos AzureDiagnostics, pero se agregan a la consulta analizando el resultado de la columna "Mensaje" para simplificar su análisis.

Sugerencias para solución de problemas:

- Para identificar el inicio de una negociación IPSec, debe buscar el mensaje inicial de SA\_INIT. Dicho mensaje podría enviarse por cualquier lado del túnel. Quien envía el primer paquete se denomina "iniciador" en la terminología de IPsec, mientras que el otro lado se convierte en el "respondedor". El primer mensaje SA\_INIT es siempre el que rCookie = 0.

- Si no se puede establecer el túnel IPsec, Azure lo seguirá intentando cada pocos segundos. Por esta razón, la solución de problemas de "VPN inactiva" es muy conveniente en IKEdiagnosticLog, ya que no es necesario esperar a que se produzca un tiempo específico para reproducir el problema. Además, el error en teoría siempre será el mismo cada vez que se intenta, por lo que solo se puede ampliar una negociación de error de "ejemplo" en cualquier momento.

- SA\_INIT contiene los parámetros de IPsec que el homólogo desea usar para esta negociación de IPSec. Documento oficial   
El artículo [Parámetros de IPSec/IKE predeterminados](./vpn-gateway-about-vpn-devices.md#ipsec) muestra los parámetros de IPsec que admite la puerta de enlace de Azure con la configuración predeterminada.


## <a name="p2sdiagnosticlog"></a><a name="P2SDiagnosticLog"></a>P2SDiagnosticLog

La última tabla disponible para los diagnósticos de VPN es **P2SDiagnosticLog**. En esta tabla se realiza un seguimiento de la actividad de punto a sitio.

Aquí tiene una consulta de ejemplo como referencia.

```
AzureDiagnostics  
| where Category == "P2SDiagnosticLog"  
| project TimeGenerated, OperationName, Message, Resource, ResourceGroup
```

Esta consulta en **P2SDiagnosticLog** mostrará varias columnas.

|***Nombre** _ | _ *_Descripción_** |
|---        | ---               |
|**TimeGenerated** | La marca de tiempo de cada evento, en la zona horaria UTC.|
|**OperationName** | El evento que se ha producido. Será *P2SLogEvent*.|
| **Mensaje** | El detalle de la operación que está ocurriendo.|

La salida mostrará todas las opciones de configuración de punto a sitio que ha aplicado la puerta de enlace, así como las directivas IPsec vigentes.

:::image type="content" source="./media/troubleshoot-vpn-with-azure-diagnostics/image-28-p2s-log-event.png" alt-text="Ejemplo de conexión de punto a sitio que se ha detectado en P2SDiagnosticLog.":::

Además, siempre que un cliente se conecte a través de IKEv2 o OpenVPN de punto a sitio, la tabla registrará la actividad de los paquetes, las conversaciones EAP/RADIUS y los resultados correctos y erróneos por usuario.

:::image type="content" source="./media/troubleshoot-vpn-with-azure-diagnostics/image-29-eap.png" alt-text="Ejemplo de autenticación EAP que se ha detectado en P2SDiagnosticLog":::

## <a name="next-steps"></a>Pasos siguientes

Para configurar alertas en los registros de recursos de túnel, consulte [Configuración de alertas en los registros de diagnóstico de VPN Gateway](vpn-gateway-howto-setup-alerts-virtual-network-gateway-log.md).