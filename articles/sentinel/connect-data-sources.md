---
title: Conexión de orígenes de datos a Azure Sentinel| Microsoft Docs
description: Obtenga información sobre cómo conectar orígenes de datos como Microsoft 365 Defender (anteriormente Protección contra amenazas de Microsoft), Microsoft 365 y Office 365, Azure AD, ATP y Cloud App Security a Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/01/2020
ms.author: yelevin
ms.openlocfilehash: 936789ef292ac6d06b9fd3ac1e9a81219f39e4ef
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107307833"
---
# <a name="connect-data-sources"></a>Conexión con orígenes de datos

Una vez que haya habilitado Azure Sentinel, lo primero que debe hacer es conectar los orígenes de datos. Azure Sentinel incluye varios conectores para soluciones de Microsoft, que están disponibles inmediatamente y proporcionan integración en tiempo real, entre las que se incluyen las soluciones de Microsoft 365 Defender (anteriormente Protección contra amenazas de Microsoft) y orígenes de Microsoft 365, (incluido Office 365), Azure AD, Microsoft Defender for Identity (anteriormente, Azure ATP) y Microsoft Cloud App Security, entre otros. Además, hay conectores integrados al amplio ecosistema de seguridad para soluciones que no son de Microsoft. También puede usar el formato de evento común (CEF), Syslog o la API REST para conectar los orígenes de datos con Azure Sentinel.

1. En el menú, seleccione **Data connectors** (Conectores de datos). Esta página permite ver la lista completa de los conectores que Azure Sentinel proporciona, y su estado. Seleccione el conector al que desea conectarse y seleccione **Open connector page** (Abrir la página del conector). 

   ![Galería de conectores de datos](./media/collect-data/collect-data-page.png)

1. En la página específica del conector, asegúrese de que cumple todos los requisitos previos y siga las instrucciones para conectar los datos a Azure Sentinel. Los registros pueden tardar algún tiempo en iniciar la sincronización con Azure Sentinel. Después de conectarse, verá un resumen de los datos en el gráfico **Datos recibidos**, y el estado de conectividad de los tipos de datos.

   ![Configuración de conectores de datos](./media/collect-data/opened-connector-page.png)
  
1. Haga clic en la pestaña **Siguientes pasos** para obtener una lista del contenido que Azure Sentinel proporciona para el tipo de datos específico.

   ![Pasos siguientes para los conectores](./media/collect-data/data-insights.png)
 

## <a name="data-connection-methods"></a>Métodos de conexión de datos

Los siguientes métodos de conexión de datos son compatibles con Azure Sentinel:

- **Integración de servicio a servicio**:<br> Algunos servicios se conectan de forma nativa, como los servicios de AWS y Microsoft, y aprovechan la base de Azure de la integración lista para usar. Con solo unos clics, se pueden conectar las soluciones siguientes:
    - [Amazon Web Services: CloudTrail](connect-aws.md)
    - [Azure Active Directory](connect-azure-active-directory.md): registros de auditoría y registros de inicio de sesión
    - [Azure Activity](connect-azure-activity.md) (Actividad de Azure)
    - [Azure AD Identity Protection](connect-azure-ad-Identity-protection.md)
    - [Azure DDoS Protection](connect-azure-ddos-protection.md)
    - [Azure Defender para IoT](connect-asc-iot.md) (anteriormente conocido como Azure Security Center para IoT)
    - [Azure Information Protection](connect-azure-information-protection.md)
    - [Azure Firewall](connect-azure-firewall.md)
    - [Azure Security Center](connect-azure-security-center.md): alertas desde soluciones de Azure Defender
    - [Azure Web Application Firewall (WAF)](connect-azure-waf.md) (anteriormente conocido como Microsoft WAF)
    - [Cloud App Security](connect-cloud-app-security.md)
    - [Servidor de nombres de dominio](connect-dns.md)
    - [Microsoft 365 Defender](connect-microsoft-365-defender.md): incluye incidentes de M365D y datos sin procesar de MDE
    - [Microsoft Defender para Endpoint](connect-microsoft-defender-advanced-threat-protection.md) (anteriormente Protección contra amenazas avanzada de Microsoft)
    - [Microsoft Defender for Identity](connect-azure-atp.md) (anteriormente Azure ATP)
    - [Microsoft Defender para Office 365](connect-office-365-advanced-threat-protection.md) (anteriormente conocido como Protección contra amenazas avanzada de Office 365)
    - [Office 365](connect-office-365.md) (ahora con Teams)
    - [Firewall de Windows](connect-windows-firewall.md)
    - [Eventos de seguridad de Windows](connect-windows-security-events.md)

- **Soluciones externas mediante API**: algunos orígenes de datos se conectan mediante las API proporcionadas por el origen de datos conectado. Normalmente, la mayoría de las tecnologías de seguridad proporcionan un conjunto de API a través del cual se pueden recuperar registros de eventos. Las API se conectan a Azure Sentinel y recopilan y envían tipos de datos específicos a Azure Log Analytics. Entre los dispositivos conectados mediante API se incluyen:
    
    - [Agari Phishing Defense y Agari Brand Protection](connect-agari-phishing-defense.md)
    - [Alcide kAudit](connect-alcide-kaudit.md)
    - [Barracuda WAF](connect-barracuda.md)
    - [Barracuda CloudGen Firewall](connect-barracuda-cloudgen-firewall.md)
    - [BETTER Mobile Threat Defense](connect-better-mtd.md)
    - [Beyond Security beSECURE](connect-besecure.md)
    - [Cisco Umbrella](connect-cisco-umbrella.md)
    - [Citrix Analytics (Security)](connect-citrix-analytics.md)
    - [F5 BIG-IP](connect-f5-big-ip.md)
    - [Forcepoint DLP](connect-forcepoint-dlp.md)
    - [Google Workspace (anteriormente, G Suite)](connect-google-workspace.md)
    - [Registros DNS de NXLog (Windows)](connect-nxlog-dns.md)
    - [LinuxAudit de NXLog](connect-nxlog-linuxaudit.md)
    - [SSO de Okta](connect-okta-single-sign-on.md)
    - [Orca Security](connect-orca-security-alerts.md)
    - [Registros de Perimeter 81](connect-perimeter-81-logs.md)
    - [Proofpoint On Demand (POD) Email Security](connect-proofpoint-pod.md)
    - [Proofpoint TAP](connect-proofpoint-tap.md)
    - [Qualys VM](connect-qualys-vm.md)
    - [Servicio de Salesforce en la nube](connect-salesforce-service-cloud.md)
    - [Sophos Cloud Optix](connect-sophos-cloud-optix.md)
    - [Squadra Technologies secRMM](connect-squadra-secrmm.md)
    - [Symantec ICDX](connect-symantec.md)
    - [VMware Carbon Black Cloud Endpoint Standard](connect-vmware-carbon-black.md)
    - [Zimperium](connect-zimperium-mtd.md)


- **Soluciones externas mediante agente**: Azure Sentinel se puede conectar a través de un agente a cualquier otro origen de datos que pueda realizar streaming de registro en tiempo real mediante el protocolo de Syslog.

    La mayoría de las aplicaciones usan el protocolo de Syslog para enviar mensajes de eventos que incluyen el propio registro y datos sobre este. El formato de los registros varía, pero la mayoría de los dispositivos admite el formato basado en CEF para los datos de registro. 

    El agente de Azure Sentinel, que en realidad es el agente de Log Analytics, convierte los registros con formato CEF a un formato que Log Analytics puede ingerir. Dependiendo del tipo de dispositivo, el agente se instala directamente en el dispositivo o en reenviador de registros Linux dedicado. El agente para Linux recibe eventos del demonio de Syslog a través de UDP; sin embargo,si se espera que una máquina Linux recopile un gran volumen de eventos Syslog, se envían a través de TCP desde el demonio de Syslog al agente y desde allí a Log Analytics.

    - **Firewalls, servidores proxy y puntos de conexión - CEF:**
        - [AI Vectra Detect](connect-ai-vectra-detect.md)
        - [Akamai Security Events](connect-akamai-security-events.md)
        - [Aruba ClearPass](connect-aruba-clearpass.md)
        - [Broadcom Symantec DLP](connect-broadcom-symantec-dlp.md)
        - [Check Point](connect-checkpoint.md)
        - [Cisco ASA](connect-cisco.md)
        - [Citrix WAF](connect-citrix-waf.md)
        - [CyberArk Enterprise Password Vault](connect-cyberark.md)
        - [Reveal(x) de ExtraHop](connect-extrahop.md)
        - [F5 ASM](connect-f5.md)
        - [Productos de Forcepoint](connect-forcepoint-casb-ngfw.md)
        - [Fortinet](connect-fortinet.md)
        - [AMS de Illusive Networks](connect-illusive-attack-management-system.md)
        - [Imperva WAF Gateway](connect-imperva-waf-gateway.md)
        - [One Identity Safeguard](connect-one-identity.md)
        - [Palo Alto Networks](connect-paloalto.md)
        - [Thycotic Secret Server](connect-thycotic-secret-server.md)
        - [Trend Micro Deep Security](connect-trend-micro.md)
        - [Trend Micro TippingPoint](connect-trend-micro-tippingpoint.md)
        - [Plataforma de análisis forense de redes de WireX](connect-wirex-systems.md)
        - [Zscaler](connect-zscaler.md)
        - [Otros dispositivos basados en CEF](connect-common-event-format.md)
    - **Firewalls, servidores proxy y puntos de conexión - Syslog:**
        - [Alsid para Active Directory](connect-alsid-active-directory.md)
        - [Cisco Meraki](connect-cisco-meraki.md)
        - [Cisco Unified Computing System (UCS)](connect-cisco-ucs.md)
        - [Infoblox NIOS](connect-infoblox.md)
        - [Juniper SRX](connect-juniper-srx.md)
        - [Pulse Connect Secure](connect-pulse-connect-secure.md)
        - [Sophos XG](connect-sophos-xg-firewall.md)
        - [Squid Proxy](connect-squid-proxy.md)
        - [Symantec ProxySG](connect-symantec-proxy-sg.md)
        - [Symantec VIP](connect-symantec-vip.md)
        - [VMware ESXi](connect-vmware-esxi.md)
        - [Otros dispositivos basados en Syslog](connect-syslog.md)
    - [Servidor HTTP de Apache](connect-apache-http-server.md)
    - Soluciones de DLP
    - [Threat intelligence providers](connect-threat-intelligence.md) (Proveedores de información sobre amenazas)
    - [DNS machines](connect-dns.md) (Máquinas DNS): agente instalado directamente en la máquina DNS
    - [Máquinas virtuales de Azure Stack](connect-azure-stack.md)
    - Servidores Linux
    - Otras nubes
    
## <a name="agent-connection-options"></a>Opciones de conexión del agente<a name="agent-options"></a>

Para conectar su dispositivo externo a Azure Sentinel, el agente debe implementarse en una máquina dedicada (máquina virtual o local) para admitir la comunicación entre el dispositivo y Azure Sentinel. Puede implementar el agente automáticamente o de forma manual. La implementación automática solo está disponible si su máquina dedicada es una nueva máquina virtual que crea en Azure. 

![CEF en Azure](./media/connect-cef/cef-syslog-azure.png)

También puede implementar el agente manualmente en una máquina virtual existente, en una máquina virtual en otra nube o en una máquina local.

![CEF local](./media/connect-cef/cef-syslog-onprem.png)

## <a name="map-data-types-with-azure-sentinel-connection-options"></a>Asignación de tipos de datos con opciones de conexión de Azure Sentinel


| **Tipo de datos** | **Conexión** | **¿Conector de datos?** | **Comentarios** |
|------|---------|-------------|------|
| AWSCloudTrail | [Conexión de AWS](connect-aws.md) | &#10003; | |
| AzureActivity | [Conexión del registro de actividad de Azure](connect-azure-activity.md) e [Introducción a los registros de actividad](../azure-monitor/essentials/platform-logs-overview.md)| &#10003; | |
| AuditLogs | [Conexión de Azure AD](connect-azure-active-directory.md)  | &#10003; | |
| SigninLogs | [Conexión de Azure AD](connect-azure-active-directory.md)  | &#10003; | |
| AzureFirewall |[Diagnóstico de Azure](../firewall/firewall-diagnostics.md) | &#10003; | |
| InformationProtectionLogs_CL  | [Informes de Azure Information Protection](/azure/information-protection/reports-aip)<br>[Conexión de Azure Information Protection](connect-azure-information-protection.md)  | &#10003; | Normalmente usa la función **InformationProtectionEvents** además del tipo de datos. Para más información, consulte [Modificación de informes y creación de consultas personalizadas](/azure/information-protection/reports-aip#how-to-modify-the-reports-and-create-custom-queries)|
| AzureNetworkAnalytics_CL  | [Esquema de análisis de tráfico](../network-watcher/traffic-analytics.md) [Análisis de tráfico](../network-watcher/traffic-analytics.md)  | | |
| CommonSecurityLog  | [Conexión de CEF](connect-common-event-format.md)  | &#10003; | |
| OfficeActivity | [Conexión de Office 365](connect-office-365.md) | &#10003; | |
| SecurityEvents | [Conexión de eventos de seguridad de Windows](connect-windows-security-events.md)  | &#10003; | Para los libros de protocolos poco seguros, vea [Configuración de libros de protocolos poco seguros](./quickstart-get-visibility.md#use-built-in-workbooks)  |
| syslog | [Conexión de Syslog](connect-syslog.md) | &#10003; | |
| Firewall de aplicaciones web (WAF) de Microsoft: (AzureDiagnostics) |[Conexión del firewall de aplicaciones web de Microsoft](./connect-azure-waf.md) | &#10003; | |
| SymantecICDx_CL | [Conexión de Symantec](connect-symantec.md) | &#10003; | |
| ThreatIntelligenceIndicator  | [Conexión de Inteligencia sobre amenazas](connect-threat-intelligence.md)  | &#10003; | |
| VMConnection <br> ServiceMapComputer_CL<br> ServiceMapProcess_CL|  [Mapa de servicio de Azure Monitor](../azure-monitor/vm/service-map.md)<br>[Incorporación de VM Insights de Azure Monitor](../azure-monitor/vm/vminsights-enable-overview.md) <br> [Habilitación de VM Insights de Azure Monitor](../azure-monitor/vm/vminsights-enable-overview.md) <br> [Uso de la incorporación de una máquina virtual individual](../azure-monitor/vm/vminsights-enable-portal.md)<br>  [Uso de la incorporación mediante Policy](../azure-monitor/vm/vminsights-enable-policy.md)| &#10007; | Libro de VM Insights  |
| DnsEvents | [Conexión de DNS](connect-dns.md) | &#10003; | |
| W3CIISLog | [Conexión de registros de IIS](../azure-monitor/agents/data-sources-iis-logs.md)  | &#10007; | |
| WireData | [Conexión de Wire Data](../azure-monitor/insights/wire-data.md) | &#10007; | |
| WindowsFirewall | [Conexión de Firewall de Windows](connect-windows-firewall.md) | &#10003; | |
| AADIP SecurityAlert  | [Conexión de Azure AD Identity Protection](connect-azure-ad-identity-protection.md)  | &#10003; | |
| AATP SecurityAlert  | [Conexión de Microsoft Defender for Identity](connect-azure-atp.md) (anteriormente Azure ATP) | &#10003; | |
| ASC SecurityAlert  | [Conexión de alertas de Azure Defender](connect-azure-security-center.md) desde Azure Security Center  | &#10003; | |
| MCAS SecurityAlert  | [Conexión de Microsoft Cloud App Security](connect-cloud-app-security.md)  | &#10003; | |
| SecurityAlert | | | |
| Sysmon (evento) | [Conexión de Sysmon](https://azure.microsoft.com/blog/detecting-in-memory-attacks-with-sysmon-and-azure-security-center)<br> [Conexión de eventos de Windows](../azure-monitor/agents/data-sources-windows-events.md) <br> [Obtención del analizador de Sysmon](https://github.com/Azure/Azure-Sentinel/blob/master/Parsers/Sysmon/Sysmon-v10.42-Parser.txt)| &#10007; | La colección de Sysmon no se instala de forma predeterminada en máquinas virtuales. Para más información sobre cómo instalar el agente Sysmon, consulte [Sysmon](/sysinternals/downloads/sysmon). |
| ConfigurationData  | [Automatización del inventario de máquinas virtuales](../automation/change-tracking/overview.md)| &#10007; | |
| ConfigurationChange  | [Automatización del seguimiento de VM](../automation/change-tracking/overview.md) | &#10007; | |
| F5 BIG-IP | [Conexión de F5 BIG-IP](https://devcentral.f5.com/s/articles/Integrating-the-F5-BIGIP-with-Azure-Sentinel)  | &#10003; | |
| McasShadowItReporting  |  | &#10007; | |
| Barracuda_CL | [Conexión de Barracuda](connect-barracuda.md) | &#10003; | |


## <a name="next-steps"></a>Pasos siguientes

- Para empezar a trabajar con Azure Sentinel, necesita una suscripción a Microsoft Azure. Si no tiene una suscripción, puede registrarse para obtener una [evaluación gratuita](https://azure.microsoft.com/free/).
- Aprenda a [incorporar los datos en Azure Sentinel](quickstart-onboard.md), [obtenga visibilidad sobre ellos y aprenda a defenderse de posibles amenazas](quickstart-get-visibility.md).
