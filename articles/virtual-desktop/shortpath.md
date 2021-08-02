---
title: RDP Shortpath de Azure Virtual Desktop (versión preliminar)
titleSuffix: Azure
description: Configuración de RDP Shortpath (versión preliminar) para Azure Virtual Desktop.
author: gundarev
ms.topic: conceptual
ms.date: 11/16/2020
ms.author: denisgun
ms.openlocfilehash: bf4612afa0e75d4a63a13358095027c68c93e6da
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111755879"
---
# <a name="azure-virtual-desktop-rdp-shortpath-preview"></a>RDP Shortpath de Azure Virtual Desktop (versión preliminar)

> [!IMPORTANT]
> Actualmente, RDP Shortpath está en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

RDP Shortpath es una característica de Azure Virtual Desktop que establece un transporte basado en UDP directo entre el cliente de Escritorio remoto y el host de sesión. RDP usa este transporte para proporcionar Escritorio remoto y RemoteApp, a la vez que ofrece una mejor confiabilidad y una latencia coherente.

## <a name="key-benefits"></a>Ventajas principales

* El transporte de RDP Shortpath se basa en el [protocolo de control de velocidad universal (URCP)](https://www.microsoft.com/en-us/research/publication/urcp-universal-rate-control-protocol-for-real-time-communication-applications/). URCP mejora UDP con la supervisión activa de las condiciones de la red y proporciona un uso equitativo y completo de los vínculos. URCP funciona con niveles de retraso y pérdida bajos, según las necesidades de Escritorio remoto. URCP logra el mejor rendimiento al aprender dinámicamente los parámetros de red y proporcionar un protocolo con un mecanismo de control de velocidad.
* RDP Shortpath establece la conectividad directa entre el cliente de Escritorio remoto y el host de sesión. La conectividad directa reduce la dependencia de las puertas de enlace de Azure Virtual Desktop, mejora la confiabilidad de la conexión y aumenta el ancho de banda disponible para cada sesión de usuario.
* La eliminación de la retransmisión adicional reduce el tiempo de ida y vuelta, de modo que se mejora la experiencia del usuario con aplicaciones y métodos de entrada sensibles a la latencia.
* RDP Shortpath permite [configurar la prioridad de la calidad de servicio (QoS)](./rdp-quality-of-service-qos.md) para las conexiones RDP a través de marcas de punto de código de servicios diferenciados (DSCP).
* El transporte de RDP Shortpath permite [limitar el tráfico de red saliente](./rdp-bandwidth.md#limit-network-bandwidth-use-with-throttle-rate) mediante la especificación de una tasa de limitación para cada sesión.

## <a name="connection-security"></a>Seguridad de conexión

RDP Shortpath amplía las capacidades multitransporte de RDP. No reemplaza el transporte de conexión inversa, sino que lo complementa. Toda la administración de sesiones inicial se realiza a través de la infraestructura de Azure Virtual Desktop.

El puerto UDP 3390 solo se usa para el tráfico de Shortpath entrante, que se autentica a través del transporte de conexión inversa. El agente de escucha de RDP Shortpath omite todos los intentos de conexión con el agente de escucha, a menos que coincidan con la sesión de conexión inversa.

RDP Shortpath usa una conexión TLS entre el cliente y el host de sesión a través de los certificados del host de sesión. De forma predeterminada, el certificado usado para el cifrado RDP lo genera automáticamente el sistema operativo durante la implementación. Si lo desea, los clientes pueden implementar certificados administrados centralmente emitidos por la entidad de certificación de empresa. Para más información sobre las configuraciones de certificados, consulte la [documentación de Windows Server](/troubleshoot/windows-server/remote/remote-desktop-listener-certificate-configurations).

## <a name="rdp-shortpath-connection-sequence"></a>Secuencia de conexión de RDP Shortpath

Después de instalar el [transporte de conexión inversa](./network-connectivity.md), el cliente y el host de sesión establecen la conexión RDP y negocian las funcionalidades multitransporte. Los pasos adicionales se describen a continuación:

1. El host de sesión envía la lista de sus direcciones IPv4 e IPv6 privadas y públicas al cliente.
2. El cliente inicia el subproceso en segundo plano para establecer un transporte paralelo basado en UDP directamente en una de las direcciones IP del host.
3. Mientras el cliente está sondeando las direcciones IP proporcionadas, este continúa el establecimiento de la conexión inicial a través del transporte de conexión inversa para garantizar que no haya ningún retraso en la conexión del usuario.
4. Si el cliente tiene una línea de visión directa y la configuración del firewall es correcta, el cliente establece una conexión TLS segura con el host de sesión.
5. Después de establecer el transporte de Shortpath, RDP mueve todos los canales virtuales dinámicos (DVCs) al nuevo transporte, incluidos los gráficos remotos, la entrada y el redireccionamiento del dispositivo.
6. Si un firewall o una topología de red impide que el cliente establezca conectividad UDP directa, RDP continúa con un transporte de conexión inversa.

En el diagrama siguiente se ofrece información general de la conexión de red de RDP Shortpath.

:::image type="content" source="media/rdp-shortpath-connections.svg" alt-text="Diagrama de conexiones de red de RDP Shortpath" lightbox="media/rdp-shortpath-connections.svg":::

## <a name="requirements"></a>Requisitos

Para admitir RDP Shortpath, el cliente de Azure Virtual Desktop necesita una línea de visión directa hacia el host de sesión. Puede obtener una línea de visión directa mediante una de las siguientes tecnologías:

* Los equipos cliente remotos deben ejecutar Windows 10 o Windows 7 y tener instalado el [cliente de Escritorio de Windows](/windows-server/remote/remote-desktop-services/clients/windowsdesktop). Actualmente, no se admite el cliente web.
* [Emparejamiento privado de ExpressRoute](../expressroute/expressroute-circuit-peerings.md)
* [VPN de sitio a sitio (basada en IPsec)](../vpn-gateway/tutorial-site-to-site-portal.md)
* [VPN de punto a sitio (basada en IPsec)](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md)
* [Asignación de dirección IP pública](../virtual-network/virtual-network-public-ip-address.md)

Si usa otros tipos de VPN para conectarse a la red virtual de Azure, se recomienda usar una VPN basada en UDP para obtener los mejores resultados. Aunque la mayoría de las soluciones VPN basadas en TCP encapsulan todos los paquetes de IP, incluidos los UDP, agregan una sobrecarga heredada del control de congestiones de TCP que ralentizaría el rendimiento de RDP.

La línea de visión directa implica que los firewalls no bloquean el puerto UDP 3390 y el cliente puede conectarse directamente al host de sesión.

## <a name="enabling-rdp-shortpath-preview"></a>Habilitación de la versión preliminar de RDP Shortpath

Para participar en la versión preliminar de RDP Shortpath, debe habilitar el agente de escucha de RDP Shortpath en el host de sesión. Puede habilitar RDP Shortpath en tantos hosts de sesión como se usen en su entorno. No es necesario habilitar RDP Shortpath en todos los hosts del grupo.
Para habilitar el agente de escucha de Shortpath, debe configurar los siguientes valores del Registro:

> [!WARNING]
> Pueden ocurrir problemas serios si modifica el Registro incorrectamente mediante el Editor del Registro u otro método. Estos problemas pueden requerir que vuelva a instalar el sistema operativo. Microsoft no brinda ninguna garantía de que se puedan solucionar estos problemas. Modifique el registro bajo su responsabilidad.

1. En el host de sesión, inicie regedit.exe y vaya a la siguiente ubicación:

    ```cmd
    HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations
    ```

2. Cree un nuevo valor **DWORD** denominado **fUseUdpPortRedirector** y establézcalo en **1** (decimal).
3. Cree un nuevo valor **DWORD** denominado **UdpPortNumber** y establézcalo en **3390** (decimal).
4. Salga del Editor del Registro.
5. Reinicie al host de sesión.

También puede ejecutar los siguientes cmdlets en una ventana de PowerShell con privilegios elevados para establecer los valores del registro:

```powershell
$WinstationsKey = 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations'
New-ItemProperty -Path $WinstationsKey -Name 'fUseUdpPortRedirector' -ErrorAction:SilentlyContinue -PropertyType:dword -Value 1 -Force
New-ItemProperty -Path $WinstationsKey -Name 'UdpPortNumber' -ErrorAction:SilentlyContinue -PropertyType:dword -Value 3390 -Force
```

También puede usar PowerShell para configurar la directiva de grupo.

```powershell
# Replace $domainName value with the name of your Active Directory domain
# Replace $policyName value with the name of existing Group Policy Object
$domainName = "contoso.com"
$policyName = "RDP Shortpath Policy"
Set-GPPrefRegistryValue -Key 'HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations' -ValueName 'fUseUdpPortRedirector' -Value 1 -Type:DWord  -Action:Create -Context:Computer -Name $policyName -Domain $domainName
Set-GPPrefRegistryValue -Key 'HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations' -ValueName 'UdpPortNumber' -Value 3390 -Type:DWord  -Action:Create -Context:Computer -Name $policyName -Domain $domainName
```

## <a name="configure-windows-defender-firewall-with-advanced-security"></a>Configuración del Firewall de Windows Defender con seguridad avanzada

Para permitir el tráfico de red entrante para RDP Shortpath, use el nodo Firewall de Windows Defender con seguridad avanzada en el complemento MMC de administración de directivas de grupo para crear reglas de firewall.

1. Abra la consola de administración de directivas de grupo en [Firewall de Windows Defender con seguridad avanzada](/windows/security/threat-protection/windows-firewall/open-the-group-policy-management-console-to-windows-firewall-with-advanced-security).
2. En el panel de navegación, seleccione **Reglas de entrada**.
3. Seleccione **Acción** y, después, **Nueva regla**.
4. En la página **Tipo de regla** del Asistente para nueva regla de entrada, seleccione **Personalizada** y, después, seleccione **Siguiente**.
5. En la página **Programa**, seleccione **Esta ruta de acceso del programa** y escriba "%SystemRoot%\system32\svchost.exe" y, a continuación, seleccione **Siguiente**.
6. En la página **Protocolo y puertos**, seleccione el tipo de protocolo UDP. En **Puerto Local**, seleccione "Puertos específicos" y escriba "3390".
7. En la página **Ámbito**, puedes especificar que la regla se aplica solo al tráfico de red hacia o desde las direcciones IP especificadas en esta página. Realice la configuración según su diseño y después seleccione **Siguiente**.
8. En la página **Acción**, seleccione **Permitir la conexión** y, a continuación, **Siguiente**.
9. En la página **Perfil**, seleccione los tipos de ubicación de red a los que se aplicará esta regla y luego seleccione **Siguiente**.
10. En la página **Nombre**, escribe un nombre y una descripción para la regla y luego selecciona **Finalizar**.

Puede comprobar que la nueva regla coincide con las siguientes capturas de pantallas: :::image type="content" source="media/rdp-shortpath-firewall-general-tab.png" alt-text="Captura de pantalla de la pestaña General de la configuración del firewall para las conexiones de red de RDP Shortpath" lightbox="media/rdp-shortpath-firewall-general-tab.png":::

:::image type="content" source="media/rdp-shortpath-firewall-service-settings.png" alt-text="Captura de pantalla de la pestaña Programas y servicios de la configuración del firewall para las conexiones de red de RDP Shortpath" lightbox="media/rdp-shortpath-firewall-service-settings.png":::

:::image type="content" source="media/rdp-shortpath-firewall-protocol-and-ports.png" alt-text="Captura de pantalla de la pestaña Protocolos y puertos de la configuración del firewall para las conexiones de red de RDP Shortpath" lightbox="media/rdp-shortpath-firewall-protocol-and-ports.png":::

También puede usar PowerShell para configurar el firewall de Windows:

```powershell
New-NetFirewallRule -DisplayName 'Remote Desktop - Shortpath (UDP-In)'  -Action Allow -Description 'Inbound rule for the Remote Desktop service to allow RDP traffic. [UDP 3390]' -Group '@FirewallAPI.dll,-28752' -Name 'RemoteDesktop-UserMode-In-Shortpath-UDP'  -PolicyStore PersistentStore -Profile Domain, Private -Service TermService -Protocol udp -LocalPort 3390 -Program '%SystemRoot%\system32\svchost.exe' -Enabled:True
```

### <a name="using-powershell-to-configure-windows-defender-firewall"></a>Uso de PowerShell para configurar el Firewall de Windows Defender

También puede usar PowerShell para configurar la directiva de grupo.

```powershell
# Replace $domainName value with the name of your Active Directory domain
# Replace $policyName value with the name of existing Group Policy Object
$domainName = "contoso.com"
$policyName = "RDP Shortpath Policy"
$gpoSession = Open-NetGPO -PolicyStore "$domainName\$policyName"
New-NetFirewallRule -DisplayName 'Remote Desktop - Shortpath (UDP-In)'  -Action Allow -Description 'Inbound rule for the Remote Desktop service to allow RDP traffic. [UDP 3390]' -Group '@FirewallAPI.dll,-28752' -Name 'RemoteDesktop-UserMode-In-Shortpath-UDP' -Profile Domain, Private -Service TermService -Protocol udp -LocalPort 3390 -Program '%SystemRoot%\system32\svchost.exe' -Enabled:True -GPOSession $gpoSession
Save-NetGPO -GPOSession $gpoSession
```

## <a name="configuring-azure-network-security-group"></a>Configuración de un grupo de seguridad de red de Azure

Para permitir el acceso al agente de escucha de RDP Shortpath a través de los límites de seguridad de red, debe configurar el grupo de seguridad de red de Azure para habilitar el puerto 3390 UDP entrante.
Siga las instrucciones de la [documentación del grupo de seguridad de red](../virtual-machines/windows/nsg-quickstart-portal.md) para crear una regla de seguridad de entrada que permita el tráfico con los parámetros siguientes:

* **Origen** - **Cualquiera** o el intervalo IP en el que residen los clientes
* **Intervalos de puertos de origen** -  **\***
* **Destino** - **Cualquiera**
* **Intervalos de puertos de destino** - **3390**
* **Protocolo** - **UDP**
* **Acción** - **Permitir**
* Si quiere, cambie la **Prioridad**. La prioridad afecta al orden en que se aplican las reglas (cuanto más bajo es el valor numérico, antes se aplica la regla).
* **Nombre** - - **RDP Shortpath**

### <a name="disabling-rdp-shortpath-for-a-specific-subnet"></a>Deshabilitación de RDP Shortpath para una subred específica

Si necesita bloquear subredes específicas desde el transporte de RDP Shortpath, puede configurar grupos de seguridad de red adicionales que especifiquen los intervalos IP de origen.

## <a name="verifying-the-connectivity"></a>Comprobación de la conectividad

### <a name="using-connection-information-dialog"></a>Uso del cuadro de diálogo Información sobre la conexión

Para comprobar que las conexiones están usando RDP Shortpath, abra el cuadro de diálogo "Información sobre la conexión". Para ello, haga clic en el icono de antena en la barra de herramientas de conexión.

:::image type="content" source="media/rdp-shortpath-connection-bar.png" alt-text="Imagen de la barra de conexión de Escritorio remoto":::

:::image type="content" source="media/rdp-shortpath-connection-info.png" alt-text="Imagen del cuadro de diálogo Información sobre la conexión de Escritorio remoto":::

### <a name="using-event-logs"></a>Uso de los registros de eventos

Para comprobar que la sesión usa el transporte de RDP Shortpath:

1. Conéctese al escritorio de la máquina virtual mediante el cliente de Azure Virtual Desktop.
2. Inicie el Visor de eventos y vaya hasta el siguiente nodo: **Registros de aplicaciones y servicios > Microsoft > Windows > RemoteDesktopServices-RdpCoreCDV > Microsoft-Windows-RemoteDesktopServices-RdpCoreCDV/Operational**
3. Para determinar si se está usando el transporte de RDP Shortpath, busque el id. de evento 131.

### <a name="using-log-analytics-to-verify-shortpath-connectivity"></a>Uso de Log Analytics para comprobar la conectividad de Shortpath

Si usa [Azure Log Analytics](./diagnostics-log-analytics.md), puede supervisar las conexiones al consultar la [tabla WVDConnections](/azure/azure-monitor/reference/tables/wvdconnections). Una columna denominada UdpUse indica si la pila de RDP de Azure Virtual Desktop usa el protocolo UDP en la conexión de usuario actual.
Los valores posibles son:

* **0**: la conexión del usuario no usa RDP Shortpath.
* **1**: la conexión del usuario está usando RDP Shortpath.
  
La siguiente lista de consultas le permite examinar la información de conexión. Puede ejecutar esta consulta en el [editor de consultas de Log Analytics](../azure-monitor/logs/log-analytics-tutorial.md#write-a-query). En todas las consultas, reemplace `userupn` por el UPN del usuario que desee buscar.

```kusto
let Events = WVDConnections | where UserName == "userupn" ;
Events
| where State == "Connected"
| project CorrelationId , UserName, ResourceAlias , StartTime=TimeGenerated, UdpUse, SessionHostName, SessionHostSxSStackVersion
| join (Events
| where State == "Completed"
| project EndTime=TimeGenerated, CorrelationId, UdpUse)
on CorrelationId
| project StartTime, Duration = EndTime - StartTime, ResourceAlias, UdpUse,  SessionHostName, SessionHostSxSStackVersion
| sort by StartTime asc
```

## <a name="troubleshooting"></a>Solución de problemas

### <a name="verify-shortpath-listener"></a>Comprobación del agente de escucha de Shortpath

Para comprobar que el agente de escucha de UDP está habilitado, use el siguiente comando de PowerShell en el host de sesión:

```powershell
Get-NetUDPEndpoint -OwningProcess ((Get-WmiObject win32_service -Filter "name = 'TermService'").ProcessId)  -LocalPort 3390
```

Si está habilitado, verá un resultado similar al siguiente.

```dos
LocalAddress                             LocalPort
------------                             ---------
::                                       3390
0.0.0.0                                  3390
```

Si hay un conflicto, puede identificar el proceso que está usando el puerto con el siguiente comando.

```powershell
Get-Process -id (Get-NetUDPEndpoint  -LocalPort 3390 -LocalAddress 0.0.0.0).OwningProcess
```

## <a name="disabling-rdp-shortpath"></a>Deshabilitación de RDP Shortpath

En algunos casos, quizá necesite deshabilitar el transporte de RDP Shortpath. Puede deshabilitar RDP Shortpath mediante la directiva de grupo.

### <a name="disabling-rdp-shortpath-on-the-client"></a>Deshabilitación de RDP Shortpath en el cliente

Si quiere deshabilitar RDP Shortpath para un cliente específico, puede usar la siguiente directiva de grupo para deshabilitar la compatibilidad con UDP:

1. En el cliente, ejecute **gpedit.msc**.
2. Navegue a **Configuración del equipo > Plantillas de administración > Componentes de Windows > Cliente de conexión a Escritorio remoto**.
3. Establezca la opción **"Desactivar UDP en el cliente"** en **Habilitado**.

### <a name="disabling-rdp-shortpath-on-the-session-host"></a>Deshabilitación de RDP Shortpath en el host de sesión

Si quiere deshabilitar RDP Shortpath para un host de sesión específico, puede usar la siguiente directiva de grupo para deshabilitar la compatibilidad con UDP:

1. En el host de sesión, ejecute **gpedit.msc**.
2. Navegue a **Configuración del equipo > Plantillas de administración > Componentes de Windows > Host de conexión de Escritorio remoto > Conexiones**.
3. Establezca la opción **"Seleccionar protocolos de transporte RDP"** en **Solo TCP**.

## <a name="public-preview-feedback"></a>Comentarios sobre la versión preliminar pública

Nos gustaría conocer sus experiencias con esta versión preliminar pública.
* Si tiene preguntas, solicitudes, ideas y comentarios, [use este formulario de comentarios](https://aka.ms/RDPShortpathFeedback).

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre la conectividad de red de Azure Virtual Desktop, consulte la [descripción de la conectividad de red de Azure Virtual Desktop](network-connectivity.md).
* Para empezar a trabajar con calidad de servicio (QoS) para Azure Virtual Desktop, consulte [Implementación de calidad de servicio (QoS) para Azure Virtual Desktop](rdp-quality-of-service-qos.md).
