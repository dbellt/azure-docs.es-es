---
title: Referencia del esquema del origen de datos de Azure Sentinel
description: En este artículo se enumeran los esquemas de orígenes de datos de Azure y de terceros compatibles con Azure Sentinel, con vínculos a la documentación de referencia.
author: batamig
ms.author: bagol
manager: rkarlin
ms.assetid: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: reference
ms.custom: ''
ms.date: 01/14/2021
ms.openlocfilehash: 4601f2d6eddbbe8809dfd46a7e0cc5aa3c40c722
ms.sourcegitcommit: c3739cb161a6f39a9c3d1666ba5ee946e62a7ac3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107209577"
---
# <a name="data-source-schema-reference"></a>Referencia del esquema del origen de datos

En este artículo se enumeran los esquemas de origen de datos de Azure y de terceros compatibles con vínculos a la documentación de referencia.

## <a name="azure-data-sources"></a>Orígenes de datos de Azure

| Tipo                             | Origen de datos             | Nombre de tabla de Log Analytics | Referencia de esquemas |
| -------------------------------- | ---------------------- | ---------------------- | ---------------- |
| **Azure**                            | Azure Active Directory | SigninEvents           | [Propiedades de inicio de sesión de informes de actividad de Azure AD](/graph/api/resources/signin#properties) |
| **Azure**                            | Azure Active Directory | AuditLogs              | [Referencia de AuditLogs de Azure Monitor](/azure/azure-monitor/reference/tables/auditlogs) |
| **Azure**                            | Azure Active Directory | AzureActivity          | [Referencia de AzureActivity de Azure Monitor](/azure/azure-monitor/reference/tables/azureactivity) |
| **Azure**                            | Office                 | OfficeActivity         | Esquemas de la API de actividad de administración de Office 365: <br>- [Esquema común ](/office/office-365-management-api/office-365-management-activity-api-schema#common-schema)   <br>- [Esquema de administración de Exchange ](/office/office-365-management-api/office-365-management-activity-api-schema#exchange-admin-schema) <br>- [Esquema de buzón de Exchange](/office/office-365-management-api/office-365-management-activity-api-schema#exchange-mailbox-schema)  <br>- [Esquema base de SharePoint](/office/office-365-management-api/office-365-management-activity-api-schema#sharepoint-base-schema)   <br>- [Operaciones de archivos de SharePoint](/office/office-365-management-api/office-365-management-activity-api-schema#sharepoint-file-operations) |
| **Azure**                            | Azure Key Vault         | AzureDiagnostics       | [Referencia de AzureDiagnostics de Azure Monitor](/azure/azure-monitor/reference/tables/azurediagnostics) |
| **Host**                             | Linux                  | syslog                 | [Referencia de Syslog de Azure Monitor](/azure/azure-monitor/reference/tables/syslog) |
| **Network**                          | Registros IIS               | W3CIISLog              | [Referencia de W3CIISLog de Azure Monitor](/azure/azure-monitor/reference/tables/w3ciislog) |
| **Network**                          | VMinsights             | VMConnection           | [Referencia de VMConnection de Azure Monitor](/azure/azure-monitor/reference/tables/vmconnection) |
| **Network**                          | Solución Wire Data     | WireData               | [Referencia de Azure Monitor WireData](/azure/azure-monitor/reference/tables/wiredata) |
| **Network**                          | Registro de flujo de NSG          | AzureNetworkAnalytics  | [Esquema y agregación de datos en Análisis de tráfico](../network-watcher/traffic-analytics-schema.md) |
| | | | |

> [!NOTE]
> Para obtener más información, consulte la [referencia de datos de Azure Monitor](/azure/azure-monitor/reference/) completa.
>
## <a name="3rd-party-vendor-data-sources"></a>Orígenes de datos de proveedores de terceros

En la tabla siguiente se enumeran los proveedores de terceros admitidos y su documentación de la asignación de Syslog o el formato de evento común (CEF) para los distintos tipos de registro compatibles, que contienen asignaciones de campos de CEF y registros de ejemplo para cada tipo de categoría.

| Tipo |    Vendor |    Producto | Nombre de tabla de Log Analytics | Referencia de asignación de campos de CEF  |
| ----- | ----- | ----- | ----- |----- |
| **Network** | Palo Alto   | PAN OS    | CommonSecurityLog |   [Guía de integración de formato de evento común de PAN-OS 9.0](https://docs.paloaltonetworks.com/content/dam/techdocs/en_US/pdf/cef/pan-os-90-cef-configuration-guide.pdf) (busque  *Formatos de registro de estilo CEF*) |
| **Network** | Punto de comprobación  |ALL   | CommonSecurityLog | [Descripción de los campos de registro](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk109795)       |
| **Network** | Fortigate   | ALL   | CommonSecurityLog | [Estructura del esquema de registro](https://docs.fortinet.com/document/fortigate/6.2.3/fortios-log-message-reference/738142/log-schema-structure)         |
| **Network** | Barracuda | Firewall de aplicaciones web |  CommonSecurityLog   | [Configuración de Syslog y otros registros](https://campus.barracuda.com/product/webapplicationfirewall/doc/4259935/how-to-configure-syslog-and-other-logs/)  |
| **Network** | Cisco | ASA | CommonSecurityLog | [Mensajes de Syslog de la serie Cisco ASA](https://www.cisco.com/c/en/us/td/docs/security/asa/syslog/b_syslog/about.html)    |
| **Network** | Cisco | Firepower   | CommonSecurityLog | [Mensajes de Syslog de Cisco Firepower Threat Defense](https://www.cisco.com/c/en/us/td/docs/security/firepower/Syslogs/b_fptd_syslog_guide.html)    |
| **Network** | Cisco   | Umbrella  | Tabla de registros personalizados  | [Formatos de registro y control de versiones](https://docs.umbrella.com/deployment-umbrella/docs/log-formats-and-versioning)   |
| **Network**   | Cisco | Meraki    | CommonSecurityLog |   [Ejemplos de registro y tipos de eventos de Syslog](https://documentation.meraki.com/zGeneral_Administration/Monitoring_and_Reporting/Syslog_Event_Types_and_Log_Samples)    |
| **Network**   | Zscaler | Nano Streaming Service (NSS)|   CommonSecurityLog | [Aplicación de formato a las fuentes NSS](https://help.zscaler.com/zia/documentation-knowledgebase/analytics/nss/nss-feeds/formatting-nss-feeds) (solo registros web, firewall, DNS y túnel) |
| **Network**   |F5 | BigIP LTM|    CommonSecurityLog|  [Mensajes de eventos y tipos de ataque](https://techdocs.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/bigip-external-monitoring-implementations-13-0-0/15.html)  |
| **Network** | F5  | BigIP ASM|    CommonSecurityLog|  [Registro de eventos de seguridad de la aplicación](https://techdocs.f5.com/kb/en-us/products/big-ip_asm/manuals/product/asm-implementations-13-1-0/14.html)                                                           |
| **Network** | Citrix  |Firewall de aplicación web   | CommonSecurityLog|    [Compatibilidad con el registro de formato de evento común (CEF) en el firewall de aplicación](https://support.citrix.com/article/CTX136146) <br>  [Referencia de mensajes de Syslog de NetScaler 12.0](https://developer-docs.citrix.com/projects/netscaler-syslog-message-reference/en/12.0/)   |
|**Host** |Symantec | Symantec Endpoint Protection Manager (SEPM) | CommonSecurityLog|[Configuración del registro externo y los niveles de gravedad de los eventos de registro para Endpoint Protection Manager](https://support.symantec.com/us/en/article.tech171741.html)|
|**Host** |Trend Micro |All |CommonSecurityLog | [Asignación de contenido de Syslog: CEF](https://docs.trendmicro.com/en-us/enterprise/control-manager-70/appendices/syslog-mapping-cef.aspx) |
| | | | | |

> [!NOTE]
> Para obtener más información, consulte también [asignación de campos CEF y CommonSecurityLog](cef-name-mapping.md).
> 
## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre los conectores de Azure Sentinel compatibles, como los conectores de CEF, Syslog, directos, de agente y personalizados:

- [Conexión con orígenes de datos](connect-data-sources.md)

- [Azure Sentinel Syslog, CEF y otros conectores de terceros](https://techcommunity.microsoft.com/t5/azure-sentinel/azure-sentinel-syslog-cef-and-other-3rd-party-connectors-grand/ba-p/803891)
