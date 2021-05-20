---
title: Disponibilidad de características en la nube de servicios de Azure para clientes de la Administración Pública de Estados Unidos
description: Incluya una lista de la disponibilidad de características para los servicios de seguridad de Azure, como Azure Sentinel para los clientes de la Administración Pública de Estados Unidos
author: batami
ms.author: bagol
ms.service: security
ms.topic: reference
ms.date: 04/29/2021
ms.openlocfilehash: d545d406bc914eaf634ff2c47698df7c1206a5cb
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109655523"
---
# <a name="cloud-feature-availability-for-us-government-customers"></a>Disponibilidad de las características en la nube para clientes de la Administración Pública de Estados Unidos

En este artículo se describe la disponibilidad de características en las nubes de Microsoft Azure y Azure Government para los siguientes servicios de seguridad:

- Azure Sentinel

> [!NOTE]
> Pronto se agregarán servicios de seguridad adicionales a este artículo.
> 

## <a name="azure-government"></a>Azure Government

Azure Government usa las mismas tecnologías subyacentes que Azure (a veces denominadas Azure Commercial o Azure Public), que incluye los componentes principales de la infraestructura como servicio (IaaS), la plataforma como servicio (PaaS) y el software como servicio (SaaS). Tanto Azure como Azure Government cuentan con controles de seguridad integrales y el compromiso de Microsoft para la protección de los datos de los clientes.

Azure Government es un entorno en la nube aislado físicamente que está dedicado a los gobiernos federales, estatales, locales y tribales de Estados Unidos, así como a sus asociados. Mientras que ambos entornos en la nube se evalúan y autorizan en el nivel de impacto FedRAMP High, Azure Government proporciona un nivel adicional de protección a los clientes a través de compromisos contractuales relacionados con el almacenamiento de los datos de clientes en Estados Unidos y la limitación del posible acceso a los sistemas que procesen los datos del cliente a personas seleccionadas de Estados Unidos. Estos compromisos pueden ser de interés para los clientes que usan la nube a fin de almacenar o procesar datos sujetos a las regulaciones de control de exportación de Estados Unidos, como EAR, ITAR y 10 CFR parte 810 del Departamento de Energía de Estados Unidos.

Para obtener más información acerca de Azure Government, consulte [¿Qué es Azure Government?](/azure/azure-government/documentation-government-welcome)

## <a name="microsoft-365-integration"></a>Integración de Microsoft 365

Las integraciones entre productos se basan en la interoperabilidad entre las plataformas de Azure y Office. Las ofertas hospedadas en el entorno de Azure son accesibles desde las plataformas Microsoft 365 Enterprise y Microsoft 365 Government. Office 365 y Office 365 GCC trabajan con Azure Active Directory (Azure AD) en Azure. Office 365 GCC High y Office 365 DoD se trabajan con Azure AD en Azure Government.

En el diagrama siguiente se muestra la jerarquía de nubes de Microsoft y cómo se relacionan entre sí.

:::image type="content" source="media/microsoft-365-cloud-integration.png" alt-text="Integración de nube de Microsoft 365.":::

El entorno de Office 365 GCC ayuda a los clientes a cumplir los requisitos de la Administración Pública de Estados Unidos, incluidos FedRAMP High, CJIS y la publicación 1075 de IRS. Los entornos de Office 365 GCC High y DoD admiten a los clientes que necesitan cumplir con DoD IL4/5, DFARS 7012, NIST 800-171 e ITAR.

Para obtener más información acerca de los entornos de Office 365 para la Administración Pública de Estados Unidos, consulte:

- [Office 365 GCC](/office365/servicedescriptions/office-365-platform-service-description/office-365-us-government/gcc)
- [Office 365 GCC High y DoD](/office365/servicedescriptions/office-365-platform-service-description/office-365-us-government/gcc-high-and-dod)


En las siguientes secciones se identifica cuándo un servicio tiene una integración con Microsoft 365 y la disponibilidad de características para Office 365 GCC, Office 365 GCC High y Office 365 DoD.
## <a name="azure-sentinel"></a>Azure Sentinel

Microsoft Azure Sentinel es una solución de administración de eventos de información de seguridad (SIEM) y respuesta automatizada de orquestación de seguridad (SOAR) que es escalable y nativa de la nube. Azure Sentinel ofrece análisis de seguridad inteligente e inteligencia frente a amenazas en toda la empresa, de forma que proporciona una única solución para la detección de alertas, la visibilidad de amenazas, la búsqueda proactiva y la respuesta a amenazas.

Para más información, consulte la [documentación del producto de Azure Sentinel](/azure/sentinel/overview).

En las siguientes tablas se muestra la disponibilidad de características de Azure Sentinel actuales en Azure y Azure Government.


| Característica | Azure | Azure Government  |
| ----- | ----- | ---- |
|- [Traiga su propio aprendizaje automático (BYO-ML)](/azure/sentinel/bring-your-own-ml) | Vista previa pública | Vista previa pública |
| - [Vista de incidentes entre inquilinos o áreas de trabajo](/azure/sentinel/multiple-workspace-view) |Vista previa pública | Vista previa pública |
| - [Conclusiones sobre la entidad](/azure/sentinel/enable-entity-behavior-analytics) | Vista previa pública | No disponible |
| - [Fusion](/azure/sentinel/fusion)<br>Detección avanzada de ataques de varias fases <sup>[1](#footnote1)</sup> | GA | GA |
| - [Búsqueda](/azure/sentinel/hunting) | GA | GA |
|- [Cuadernos](/azure/sentinel/notebooks) | GA | GA |
|- [Métricas de auditoría de incidentes de centro de operaciones de seguridad](/azure/sentinel/manage-soc-with-incident-metrics) | GA | GA |
|- [Listas de reproducción](https://techcommunity.microsoft.com/t5/azure-sentinel/what-s-new-watchlist-is-now-in-public-preview/ba-p/1765887) | Vista previa pública | No disponible |
| **Compatibilidad con inteligencia sobre amenazas** | | |
| - [Inteligencia sobre amenazas: conector de datos TAXII](/azure/sentinel/import-threat-intelligence)  | Vista previa pública | No disponible |
| - [Conector de datos de la plataforma de inteligencia sobre amenazas](/azure/sentinel/import-threat-intelligence)  | Vista previa pública | No disponible |
| - [Hoja de investigación de la inteligencia sobre amenazas](https://techcommunity.microsoft.com/t5/azure-sentinel/what-s-new-threat-intelligence-menu-item-in-public-preview/ba-p/1646597)  | Vista previa pública | No disponible |
| - [Detonación de direcciones URL](https://techcommunity.microsoft.com/t5/azure-sentinel/using-the-new-built-in-url-detonation-in-azure-sentinel/ba-p/996229) | GA | No disponible |
| - [Libro de inteligencia sobre amenazas](/azure/architecture/example-scenario/data/sentinel-threat-intelligence)  | GA | No disponible |
|**Compatibilidad con la detección** | | |
| - [Detección de acceso anómalo al recurso compartido de archivos de Windows](/azure/sentinel/fusion)  | Vista previa pública | No disponible |
| - [Detección de inicios de sesión anómalos mediante RDP](/azure/sentinel/connect-windows-security-events#configure-the-security-events-connector-for-anomalous-rdp-login-detection)<br>Detección de ML integrada | Vista previa pública | No disponible |
| - [Detección de inicios de sesión anómalos mediante SSH](/azure/sentinel/connect-syslog#configure-the-syslog-connector-for-anomalous-ssh-login-detection)<br>Detección de ML integrada | Vista previa pública | No disponible |
| **Conectores de servicio de Azure** | | |
| - [Registros de actividad de Azure](/azure/sentinel/connect-azure-activity)                                  |   GA           |    GA         |
| - [Azure Active Directory](/azure/sentinel/connect-azure-active-directory)                |      GA        |       GA        |
| - [Azure ADIP](/azure/sentinel/connect-azure-ad-identity-protection)                         |  GA            |        GA              |
| - [Azure DDoS Protection](/azure/sentinel/connect-azure-ddos-protection)                |     GA         |       GA               |
| - [Azure Defender](/azure/sentinel/connect-azure-security-center)                  |    GA          |        GA              |
| - [Azure Defender para IoT](/azure/sentinel/connect-asc-iot)           |       GA       |  No disponible           |
| - [Azure Firewall ](/azure/sentinel/connect-azure-firewall)                        |   GA           |        GA              |
| - [Azure Information Protection](/azure/sentinel/connect-azure-information-protection)              |     Vista previa pública         |         No disponible             |
| - [Azure Key Vault ](/azure/sentinel/connect-azure-key-vault)                           |       Vista previa pública         |         No disponible                       |
| - [Azure Kubernetes Services (AKS)](/azure/sentinel/connect-azure-kubernetes-service)           |       Vista previa pública         |         No disponible                |
| - [Instancias de Azure SQL Database](/azure/sentinel/connect-azure-sql-logs)                        |     GA         |         GA             |
| - [Firewall de aplicaciones web de Azure](/azure/sentinel/connect-azure-waf)                                  |      GA        |      GA                |
| **Conectores de Windows** | | |
| - [Firewall de Windows](/azure/sentinel/connect-windows-firewall)                                 |     GA         |   GA           |
| - [Eventos de seguridad de Windows](/azure/sentinel/connect-windows-security-events)                                  |      GA        |         GA     |
| **Conectores externos**| | |
| - [Agari Phishing Defense y Agari Brand Protection](/azure/sentinel/connect-agari-phishing-defense)       | Vista previa pública | Vista previa pública |
| - [Analista de IA de Darktrace](/azure/sentinel/connect-data-sources)                            | Vista previa pública | Vista previa pública |
| - [Detección de IA Vectra](/azure/sentinel/connect-ai-vectra-detect)                                 | Vista previa pública | Vista previa pública |
| - [Akamai Security Events](/azure/sentinel/connect-akamai-security-events)                           | Vista previa pública | Vista previa pública |
| - [Alcide kAudit](/azure/sentinel/connect-alcide-kaudit)                                   | Vista previa pública | No disponible      |
| - [Alsid para Active Directory](/azure/sentinel/connect-alsid-active-directory)                      | Vista previa pública | No disponible      |
| - [Servidor HHTP de Apache](/azure/sentinel/connect-apache-http-server)                               | Vista previa pública | No disponible      |
| - [Aruba ClearPass](/azure/sentinel/connect-aruba-clearpass)                                  | Vista previa pública | Vista previa pública |
| - [AWS](/azure/sentinel/connect-data-sources)                                             | GA             | GA             |
| - [Firewall de Barracuda CloudGen](/azure/sentinel/connect-barracuda-cloudgen-firewall)                      | GA             | GA             |
| - [Firewall de aplicación web de Barracuda](/azure/sentinel/connect-barracuda)                       | GA             | GA             |
| - [BETTER Mobile Threat Defense MTD](/azure/sentinel/connect-better-mtd)                 | Vista previa pública | No disponible      |
| - [Beyond Security beSECURE](/azure/sentinel/connect-besecure)                        | Vista previa pública | No disponible      |
| - [Blackberry CylancePROTECT](/azure/sentinel/connect-data-sources)                        | Vista previa pública | Vista previa pública |
| - [DLP de Broadcom Symantec](/azure/sentinel/connect-broadcom-symantec-dlp)                            | Vista previa pública | Vista previa pública |
| - [Check Point](/azure/sentinel/connect-checkpoint)                                      | GA             | GA             |
| - [Cisco ASA](/azure/sentinel/connect-cisco)                                        | GA             | GA             |
| - [Cisco Meraki](/azure/sentinel/connect-cisco-meraki)                                     | Vista previa pública | Vista previa pública |
| - [Cisco Umbrella](/azure/sentinel/connect-cisco-umbrella)                                   | Vista previa pública | Vista previa pública |
| - [Cisco UCS](/azure/sentinel/connect-cisco-ucs)                                        | Vista previa pública | Vista previa pública |
| - [Cisco Firepower eStreamer](/azure/sentinel/connect-data-sources)                          | Vista previa pública | Vista previa pública |
| - [Citrix Analytics WAF](/azure/sentinel/connect-citrix-waf)                             | GA             | GA             |
| - [Common Event Format (CEF)](/azure/sentinel/connect-common-event-format)                        | GA             | GA             |
| - [Eventos de CyberArk Enterprise Password Vault (EPV)](/azure/sentinel/connect-cyberark) | Vista previa pública | Vista previa pública |
| - [ESET Enterprise Inspector](/azure/sentinel/connect-data-sources)                       | Vista previa pública | No disponible      |
| - [Eset Security Management Center](/azure/sentinel/connect-data-sources)                  | Vista previa pública | No disponible      |
| - [ExtraHop Reveal(x)](/azure/sentinel/connect-extrahop)                               | GA             | GA             |
| - [F5 BIG-IP ](/azure/sentinel/connect-f5-big-ip)                                       | GA             | GA             |
| - [F5 Networks](/azure/sentinel/connect-f5)                                     | GA             | GA             |
| - [Forcepoint NGFW](/azure/sentinel/connect-forcepoint-casb-ngfw)                                  | Vista previa pública | Vista previa pública |
| - [Forepoint CASB](/azure/sentinel/connect-forcepoint-casb-ngfw)                                  | Vista previa pública | Vista previa pública |
| - [Forepoint DLP ](/azure/sentinel/connect-forcepoint-dlp)                                   | Vista previa pública | No disponible      |
| - [Common Audit de ForgeRock para CEF](/azure/sentinel/connect-data-sources)                  | Vista previa pública | Vista previa pública |
| - [Fortinet](/azure/sentinel/connect-fortinet)                                         | GA             | GA             |
| - [Google Workspace (G Suite) ](/azure/sentinel/connect-google-workspace)                      | Vista previa pública | No disponible      |
| - [Illusive Attack Management System](/azure/sentinel/connect-illusive-attack-management-system)                | Vista previa pública | Vista previa pública |
| - [Imperva WAF Gateway](/azure/sentinel/connect-imperva-waf-gateway)                             | Vista previa pública | Vista previa pública |
| - [Infoblox NIOS](/azure/sentinel/connect-infoblox)                                    | Vista previa pública | Vista previa pública |
| - [Juniper SRX](/azure/sentinel/connect-juniper-srx)                                      | Vista previa pública | Vista previa pública |
| - [Morphisec UTPP](/azure/sentinel/connect-data-sources)                                   | Vista previa pública | Vista previa pública |
| - [Netskope](/azure/sentinel/connect-data-sources)                                         | Vista previa pública | Vista previa pública |
| - [NXLog DNS para Windows](/azure/sentinel/connect-nxlog-dns)                                             | Vista previa pública | No disponible      |
| - [NXLog LinuxAudit](/azure/sentinel/connect-nxlog-linuxaudit)                                 | Vista previa pública | No disponible      |
| - [Inicio de sesión único de Okta](/azure/sentinel/connect-okta-single-sign-on)                              | Vista previa pública | Vista previa pública |
| - [Onapsis Platform](/azure/sentinel/connect-data-sources)                                 | Vista previa pública | Vista previa pública |
| - [One Identity Safeguard](/azure/sentinel/connect-one-identity)                          | GA             | GA             |
| - [Alertas de Orca Security](/azure/sentinel/connect-orca-security-alerts)                            | Vista previa pública | No disponible      |
| - [Palo Alto Networks](/azure/sentinel/connect-paloalto)                               | GA             | GA             |
| - [Registros de actividad de Perimeter 81](/azure/sentinel/connect-perimeter-81-logs)                      | GA             | No disponible      |
| - [Proofpoint On Demand Email Security](/azure/sentinel/connect-proofpoint-pod)             | Vista previa pública | No disponible      |
| - [Proofpoint TAP](/azure/sentinel/connect-proofpoint-tap)                                   | Vista previa pública | Vista previa pública |
| - [Pulse Connect Secure](/azure/sentinel/connect-proofpoint-tap)                             | Vista previa pública | Vista previa pública |
| - [Qualys Vulnerability Management](/azure/sentinel/connect-qualys-vm)                  | Vista previa pública | Vista previa pública |
| - [Salesforce Service Cloud](/azure/sentinel/connect-salesforce-service-cloud)                         | Vista previa pública | No disponible      |
| - [SonicWall Firewall ](/azure/sentinel/connect-sophos-cloud-optix)                              | Vista previa pública | Vista previa pública |
| - [Sophos Cloud Optix](/azure/sentinel/connect-sophos-cloud-optix)                               | Vista previa pública | No disponible      |
| - [Sophos XG Firewall](/azure/sentinel/connect-sophos-xg-firewall)                               | Vista previa pública | Vista previa pública |
| - [secRMM de Squadra Technologies](/azure/sentinel/connect-squadra-secrmm)               | GA             | GA             |
| - [Squid Proxy](/azure/sentinel/connect-squid-proxy)                                      | Vista previa pública | No disponible      |
| - [Symantec Integrated Cyber Defense Exchange](/azure/sentinel/connect-symantec)       | GA             | GA             |
| - [Symantec ProxySG](/azure/sentinel/connect-symantec-proxy-sg)                                | Vista previa pública | Vista previa pública |
| - [Symantec VIP](/azure/sentinel/connect-symantec-vip)                                     | Vista previa pública | Vista previa pública |
| - [Syslog](/azure/sentinel/connect-syslog)                                           | GA             | GA             |
| - [Plataformas de inteligencia sobre amenazas](/azure/sentinel/connect-threat-intelligence)                   | Vista previa pública | No disponible      |
| - [Inteligencia sobre amenazas de TAXII](/azure/sentinel/connect-threat-intelligence)                       | Vista previa pública | No disponible      |
| - [Thycotic Secret Server](/azure/sentinel/connect-thycotic-secret-server)                          | Vista previa pública | Vista previa pública |
| - [Deep Security de Trend Micro](/azure/sentinel/connect-trend-micro)                       | GA             | GA             |
| - [TippingPoint de Trend Micro](/azure/sentinel/connect-trend-micro-tippingpoint)                         | Vista previa pública | Vista previa pública |
| - [Trend Micro XDR](/azure/sentinel/connect-data-sources)                                  | Vista previa pública | No disponible      |
| - [VMware Carbon Black Endpoint Standard](/azure/sentinel/connect-vmware-carbon-black)           | Vista previa pública | Vista previa pública |
| - [VMware ESXi](/azure/sentinel/connect-vmware-esxi)                                      | Vista previa pública | Vista previa pública |
| - [WireX Network Forensics Platform](/azure/sentinel/connect-wirex-systems)                | Vista previa pública | Vista previa pública |
| - [Zimperium Mobile Threat Defense](/azure/sentinel/connect-zimperium-mtd)                  | Vista previa pública | No disponible      |
| - [Zscaler](/azure/sentinel/connect-zscaler)                                         | GA             | GA             |
| | | |


<sup><a name="footnote1" /></a>1</sup> Las detecciones de SSH y RDP no son compatibles con las nubes soberanas porque la plataforma de aprendizaje automático de Databricks no está disponible.

### <a name="microsoft-365-data-connectors"></a>Conectores de datos de Microsoft 365

Office 365 GCC trabaja con Azure Active Directory (Azure AD) en Azure. Office 365 GCC High y Office 365 DoD se trabajan con Azure AD en Azure Government.

> [!TIP]
> Asegúrese de prestar atención al entorno de Azure para comprender dónde [es posible la interoperabilidad](#microsoft-365-integration). En la tabla siguiente, la interoperabilidad que *no* es posible está marcada con un guion (-) a fin de indicar que la compatibilidad no es pertinente.
>

| Conector | Azure  | Azure Government  |
| ------------------------------------ | -------------- | ---------------------- |
| **[Dynamics 365](/azure/sentinel/connect-dynamics-365)**                               |              |                      |
| - Office 365 GCC |Vista previa pública | -|
| - Office 365 GCC High | -|No disponible |
| - Office 365 DoD |- | No disponible|
| **[Microsoft 365 Defender](/azure/sentinel/connect-microsoft-365-defender)**                             |              |                      |
| - Office 365 GCC | Vista previa pública| -|
| - Office 365 GCC High |- |No disponible |
| - Office 365 DoD |- | No disponible|
| **[Microsoft Cloud App Security (MCAS)](/azure/sentinel/connect-cloud-app-security)**                                      |              |                      |
| - Office 365 GCC | GA| -|
| - Office 365 GCC High |-|GA |
| - Office 365 DoD |- |GA |
| **[Microsoft Cloud App Security (MCAS)](/azure/sentinel/connect-cloud-app-security)** <br>Registros de Shadow IT                                  |              |                      |
| - Office 365 GCC | Vista previa pública| -|
| - Office 365 GCC High |-|Vista previa pública |
| - Office 365 DoD |- |Vista previa pública |
| **[Microsoft Cloud App Security (MCAS)](/azure/sentinel/connect-cloud-app-security)**                  <br>Alertas                    |              |                      |
| - Office 365 GCC | Vista previa pública| -|
| - Office 365 GCC High |-|Vista previa pública |
| - Office 365 DoD |- |Vista previa pública |
| **[Microsoft Defender para punto de conexión](/azure/sentinel/connect-microsoft-defender-advanced-threat-protection)**                                       |              |                      |
| - Office 365 GCC | GA|- |
| - Office 365 GCC High |- |No disponible |
| - Office 365 DoD |- | No disponible|
| **[Microsoft Defender for Identity](/azure/sentinel/connect-azure-atp)**                                        |              |                      |
| - Office 365 GCC |Vista previa pública | -|
| - Office 365 GCC High |- | No disponible |
| - Office 365 DoD |- |No disponible |
| **[Microsoft Defender para Office 365](/azure/sentinel/connect-office-365-advanced-threat-protection)**               |              |                      |
| - Office 365 GCC |Vista previa pública |- |
| - Office 365 GCC High |- |No disponible |
| - Office 365 DoD | -|No disponible |
| **[Office 365](/azure/sentinel/connect-office-365)**                                      |              |                      |
| - Office 365 GCC | GA|- |
| - Office 365 GCC High |- |GA |
| - Office 365 DoD |- |Disponibilidad general |
| | |


## <a name="next-steps"></a>Pasos siguientes

- Información sobre el [modelo de responsabilidad compartida](shared-responsibility.md), las tareas de seguridad que administra el proveedor de nube y las tareas que administra el usuario.
- Información sobre las capacidades de la [nube de Azure Government](/azure/azure-government/documentation-government-welcome) y de la seguridad y el diseño de confianza que se usa para apoyar el cumplimiento aplicable a organizaciones locales, estatales, federales y sus asociados.
- Información sobre el [plan de Office 365 Administración Pública](/office365/servicedescriptions/office-365-platform-service-description/office-365-us-government/office-365-us-government#about-office-365-government-environments).
- Información sobre el [cumplimiento en Azure](/azure/compliance/) de los estándares legales y normativos.
