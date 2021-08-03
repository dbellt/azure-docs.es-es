---
title: Directrices del servidor proxy en Azure Virtual Desktop - Azure
description: Una serie de directrices y recomendaciones para usar servidores proxy en implementaciones de Azure Virtual Desktop.
author: Heidilohr
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: helohr
ms.reviewer: denisgun
manager: femila
ms.openlocfilehash: c8df5cbcc241877e5637abd89804071b4dcbf938
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111753096"
---
# <a name="proxy-server-guidelines-for-azure-virtual-desktop"></a>Directrices del servidor proxy en Azure Virtual Desktop

En este artículo se muestra cómo usar un servidor proxy con Azure Virtual Desktop. Las recomendaciones de este artículo solo se aplican a las conexiones entre los agentes de host de sesión, el cliente y la infraestructura de Azure Virtual Desktop. En este artículo no se explica la conectividad de red para Office, Windows 10, FSLogix u otras aplicaciones de Microsoft.

## <a name="what-are-proxy-servers"></a>¿Qué son los servidores proxy?

Se recomienda omitir los servidores proxy para el tráfico de Azure Virtual Desktop. Los servidores proxy no aumentan la seguridad de Azure Virtual Desktop. Esto se debe a que el tráfico ya está cifrado. Para obtener más información sobre la seguridad de conexión, consulte [Seguridad de conexión](network-connectivity.md#connection-security). 

La mayoría de los servidores proxy no están diseñados para admitir conexiones WebSocket de larga duración y pueden afectar a la estabilidad de las conexiones. La escalabilidad del servidor proxy también provoca problemas porque Azure Virtual Desktop usa varias conexiones a largo plazo. Si usa servidores proxy, deben tener el tamaño adecuado para ejecutar estas conexiones.

Si la zona geográfica del servidor proxy está lejos del host, esta distancia provocará más latencia en las conexiones de los usuarios. Una mayor latencia significa un tiempo de conexión más lento y una experiencia del usuario deficiente, especialmente en escenarios que necesitan elementos gráficos, audio o interacciones de baja latencia con dispositivos de entrada. Si debe usar un servidor proxy, tenga en cuenta que debe colocar el servidor en la misma ubicación geográfica que el agente y el cliente de Azure Virtual Desktop.

Si configura el servidor proxy como la única ruta de acceso que debe tomar el tráfico de Azure Virtual Desktop, los datos del protocolo de escritorio remoto (RDP) se forzarán mediante el protocolo de control de transmisión (TCP) en lugar del protocolo de datagramas de usuario (UDP). Este movimiento reduce la calidad visual y la capacidad de respuesta de la conexión remota.

En resumen, no se recomienda usar servidores proxy en Azure Virtual Desktop porque provoca problemas relacionados con el rendimiento debido a la degradación de la latencia y a la pérdida de paquetes. 

## <a name="bypassing-a-proxy-server"></a>Omisión de un servidor proxy

Si las directivas de red y seguridad de la organización requieren servidores proxy para el tráfico web, puede configurar el entorno para omitir las conexiones de Azure Virtual Desktop mientras sigue enrutando el tráfico a través del servidor proxy. Sin embargo, las directivas de cada organización son únicas, por lo que algunos métodos pueden funcionar mejor para la implementación que otros. Estos son algunos métodos de configuración que puede intentar para evitar la pérdida de rendimiento y confiabilidad en el entorno:

- Etiquetas de servicio de Azure en Azure Firewall
- Omisión del servidor proxy mediante los archivos de configuración automática de proxy (.PAC)
- Lista de omisión en la configuración del proxy local 
- Uso de servidores proxy para la configuración por usuario 
- Uso de RDP Shortpath para la conexión RDP mientras se mantiene el tráfico de servicio a través del proxy 

## <a name="recommendations-for-using-proxy-servers"></a>Recomendaciones para usar servidores proxy

Algunas organizaciones requieren que todo el tráfico de usuario pase a través de un servidor proxy para el seguimiento o la inspección de paquetes. En esta sección se describe cómo se recomienda configurar el entorno en estos casos.

### <a name="use-proxy-servers-in-the-same-azure-geography"></a>Uso de servidores proxy en la misma ubicación geográfica de Azure

Cuando se usa un servidor proxy, se controla toda la comunicación con la infraestructura de Azure Virtual Desktop y se realiza la resolución DNS y el enrutamiento Anycast a la red de Azure Front Door. Si los servidores proxy están distantes o distribuidos en una zona geográfica de Azure, la resolución geográfica será menos precisa. Una resolución geográfica menos precisa significa que las conexiones se enrutarán a un clúster de Azure Virtual Desktop remoto. Para evitar este problema, use solo servidores proxy que estén geográficamente cerca de su clúster de Azure Virtual Desktop.

### <a name="use-rdp-shortpath-for-desktop-connectivity"></a>Uso de RDP Shortpath para la conectividad de escritorio

Al habilitar RDP Shortpath, los datos RDP omitirán el servidor proxy, si es posible. Omitir el servidor proxy garantiza un enrutamiento óptimo al usar el transporte UDP. Otro tipo de tráfico de Azure Virtual Desktop, como la intermediación, la orquestación y el diagnóstico seguirán pasando por el servidor proxy. 

### <a name="dont-use-ssl-termination-on-the-proxy-server"></a>No use la terminación SSL en el servidor proxy

El protocolo Capa de sockets seguros terminación (SSL) reemplaza los certificados de seguridad de los componentes de Azure Virtual Desktop por certificados generados por el servidor proxy. Esta característica de servidor proxy permite la inspección de paquetes para el tráfico HTTPS en el servidor proxy. Sin embargo, la inspección de paquetes también aumenta el tiempo de respuesta del servicio, lo que provoca que los usuarios tarden más tiempo en iniciar sesión. En escenarios de conexión inversa, la inspección de paquetes de tráfico RDP no es necesaria porque el tráfico RDP de conexión inversa es binario y usa niveles adicionales de cifrado.

Si configura el servidor proxy para que use la inspección SSL, recuerde que no puede revertir el servidor a su estado original después de que la inspección de SSL realice cambios. Si algo en el entorno de Azure Virtual Desktop deja de funcionar mientras tiene habilitada la inspección SSL, debe deshabilitar la inspección SSL e intentarlo de nuevo antes de abrir un caso de soporte técnico. La inspección SSL también puede hacer que el agente de Azure Virtual Desktop deje de funcionar porque interfiere en las conexiones de confianza entre el agente y el servicio.

### <a name="dont-use-proxy-servers-that-need-authentication"></a>No use servidores proxy que necesiten autenticación

Los componentes de Azure Virtual Desktop en el host de sesión se ejecutan en el contexto del sistema operativo, por lo que no admiten servidores proxy que requieran autenticación. Si el servidor proxy requiere autenticación, se producirá un error en la conexión.

### <a name="plan-for-the-proxy-server-network-capacity"></a>Planeación de la capacidad de red de los servidores proxy

Los servidores proxy tienen límites de capacidad. A diferencia del tráfico HTTP normal, el tráfico RDP tiene conexiones conversacionales de larga duración que son bidireccionales y consumen gran cantidad de ancho de banda. Antes de configurar un servidor proxy, póngase en contacto con el proveedor del servidor proxy e indíquele la capacidad de proceso que tiene el servidor. Asegúrese también de preguntar cuántas sesiones de proxy puede ejecutar a la vez. Después de implementar el servidor proxy, supervise cuidadosamente su uso de recursos en busca de cuellos de botella en el tráfico de Azure Virtual Desktop.

### <a name="proxy-servers-for-windows-7-session-hosts"></a>Servidores proxy para hosts de sesión de Windows 7

Los hosts de sesión que se ejecutan en Windows 7 no admiten conexiones de servidor proxy para datos RDP de conexión inversa. Si el host de sesión no puede conectarse directamente a las puertas de enlace de Azure Virtual Desktop, la conexión no funcionará.

### <a name="proxy-servers-and--teams-optimization"></a>Servidores proxy y optimización de Teams

Azure Virtual Desktop no admite servidores proxy para la optimización de Teams.

## <a name="session-host-configuration-recommendations"></a>Recomendaciones de configuración del host de sesión

Para configurar un servidor proxy de nivel de host de sesión, debe habilitar un proxy para todo el sistema. Recuerde que la configuración para todo el sistema afecta a todos los componentes y aplicaciones del sistema operativo que se ejecutan en el host de sesión. En las secciones siguientes se ofrecen recomendaciones para configurar servidores proxy para todo el sistema.
 
### <a name="use-the-web-proxy-auto-discovery-wpad-protocol"></a>Uso del protocolo de detección automática de proxy web (WPAD)

El agente de Azure Virtual Desktop intenta localizar automáticamente un servidor proxy en la red mediante el protocolo de detección automática de proxy web (WPAD). Durante un intento de ubicación, el agente busca en el servidor de nombres de dominio (DNS) un archivo denominado **wpad.domainsuffix**. Si el agente encuentra el archivo en el servidor DNS, realiza una solicitud HTTP para un archivo denominado **wpad.dat**. La respuesta se convierte en el script de configuración de proxy que elige el servidor proxy de salida.

Para configurar la red con el fin de que use la resolución DNS para WPAD, siga las instrucciones de [Detección automática de la configuración Internet Explorer 11](/internet-explorer/ie11-deploy-guide/auto-detect-settings-for-ie11). Asegúrese de que la lista de bloques de consultas globales del servidor DNS permite la resolución WPAD siguiendo las instrucciones de [Set-DnsServerGlobalQueryBlockList](/powershell/module/dnsserver/set-dnsserverglobalqueryblocklist?view=windowsserver2019-ps&preserve-view=true).

### <a name="manually-set-a-device-wide-internet-explorer-proxy"></a>Establecimiento manual de un proxy de Internet Explorer para todo el dispositivo

Puede establecer un proxy para todo el dispositivo o un archivo de configuración automática de proxy (.PAC) que se aplique a todos los usuarios interactivos de LocalSystem y NetworkService con el [CSP de proxy de red](/windows/client-management/mdm/networkproxy-csp). 

También puede configurar el servidor proxy para la cuenta de sistema local ejecutando el siguiente comando **bitsadmin,** como se muestra en el ejemplo siguiente: 

```console
bitsadmin /util /setieproxy LOCALSYSTEM AUTOSCRIPT http://server/proxy.pac 
```

## <a name="client-side-proxy-support"></a>Compatibilidad con el proxy del lado cliente

El cliente de Azure Virtual Desktop admite servidores proxy configurados con la configuración del sistema o un [CSP de proxy de red](/windows/client-management/mdm/networkproxy-csp).

### <a name="support-for-clients-running-on-windows-7"></a>Compatibilidad con clientes que se ejecutan en Windows 7

Los clientes que se ejecutan en Windows 7 no admiten conexiones de servidor proxy para datos RDP de conexión inversa. Si el cliente no puede conectarse directamente a las puertas de enlace de Azure Virtual Desktop, la conexión no funcionará.

### <a name="azure-virtual-desktop-client-support"></a>Compatibilidad con el cliente de Azure Virtual Desktop

En la tabla siguiente se muestra qué clientes de Azure Virtual Desktop admiten servidores proxy:

| Nombre del cliente | Compatibilidad de servidor proxy |
|---|---|
| Escritorio de Windows | Yes |
| Cliente web | Yes |
| Android | No |
| iOS | Sí |
| macOS | Sí |
| Tienda Windows | Yes |

Para obtener más información sobre la compatibilidad con proxy en clientes ligeros basados en Linux, consulte [Compatibilidad con clientes ligeros](linux-overview.md).

## <a name="support-limitations"></a>Limitaciones de compatibilidad

Hay muchos servicios y aplicaciones de terceros que actúan como servidor proxy. Estos servicios de terceros incluyen firewalls distribuidos de próxima generación, sistemas de seguridad web y servidores proxy básicos. No podemos garantizar que todas las configuraciones sean compatibles con Azure Virtual Desktop. Microsoft solo proporciona compatibilidad limitada para las conexiones establecidas a través de un servidor proxy. Si tiene problemas de conectividad al usar un servidor proxy, el equipo de soporte técnico de Microsoft recomienda configurar una omisión de proxy y, a continuación, intentar reproducir el problema.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo proteger la implementación de Azure Virtual Desktop, consulte nuestra [guía de seguridad](security-guide.md).
