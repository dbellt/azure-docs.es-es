---
title: Alertas de Microsoft Cloud App Security no importadas en Azure Sentinel a través de la integración de Microsoft 365 Defender | Microsoft Docs
description: En este artículo se muestran las alertas de Microsoft Cloud App Security que se deben ingerir directamente en Azure Sentinel, ya que no las recopila Microsoft 365 Defender.
services: sentinel
cloud: na
documentationcenter: na
author: yelevin
manager: rkarlin
ms.assetid: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 04/21/2021
ms.author: yelevin
ms.openlocfilehash: 0232cbde5aaddad268aa0e4725ee3587a2461800
ms.sourcegitcommit: 18cd3c1c8cc47258c6a1a04e0e03d6248c52ef24
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107992778"
---
# <a name="microsoft-cloud-app-security-alerts-not-imported-into-azure-sentinel-through-microsoft-365-defender-integration"></a>Alertas de Microsoft Cloud App Security no importadas en Azure Sentinel a través de la integración de Microsoft 365 Defender

Al igual que los demás componentes de Microsoft Defender (Defender para punto de conexión, Defender for Identity y Defender para Office 365), Microsoft Cloud App Security genera alertas recopiladas por Microsoft 365 Defender. Microsoft 365 Defender a su vez genera incidentes ingeridos por y [sincronizados con Azure Sentinel](microsoft-365-defender-sentinel-integration.md#microsoft-365-defender-incidents-and-microsoft-incident-creation-rules) cuando el conector de Microsoft 365 Defender está habilitado.

A diferencia de los otros tres componentes, **no todos los tipos de** alertas de Cloud App Security se han incorporado a Microsoft 365 Defender, por lo que si quiere los incidentes de todas las alertas de Cloud App Security en Azure Sentinel, tendrá que ajustar las reglas de análisis de creación de incidentes de Microsoft en consecuencia, para que esas alertas que se ingieren directamente en Sentinel sigan generando incidentes, mientras que las que están incorporadas a Microsoft 365 Defender no (por lo que no habrá duplicados).

## <a name="cloud-app-security-alerts-not-onboarded-to-microsoft-365-defender"></a>Alertas de Cloud App Security no incorporadas a Microsoft 365 Defender

Las siguientes alertas no se han incorporado a Microsoft 365 Defender, y las reglas de creación de incidentes de Microsoft que generan estas alertas deben seguir estando configuradas para generar incidentes.

| Nombre para mostrar de la alerta de Cloud App Security | Nombre de alerta de Cloud App Security |
|-|-|
| **Alerta de directiva de acceso** | `ALERT_CABINET_INLINE_EVENT_MATCH` |
| **La creación de actividades del registro Tráfico de detección ha superado el límite diario** | `ALERT_DISCOVERY_TRAFFIC_LOG_EXCEEDED_LIMIT` |
| **Alerta de directiva de actividad** | `ALERT_CABINET_EVENT_MATCH_AUDIT` |
| **Alerta de filtración anómala** | `ALERT_EXFILTRATION_DISCOVERY_ANOMALY_DETECTION` |
| **Cuenta en peligro** | `ALERT_COMPROMISED_ACCOUNT` |
| **Se ha detectado una alerta de vulneración de seguridad de la aplicación** | `ALERT_MANAGEMENT_DISCOVERY_BREACHED_APP` |
| **Cuenta inactiva** | `ALERT_ZOMBIE_USER` |
| **Aumento de la puntuación de prioridad de investigación** | `ALERT_UEBA_INVESTIGATION_PRIORITY_INCREASE` |
| **Consentimiento relativo a una aplicación de OAuth malintencionada** | `ALERT_CABINET_APP_PERMISSION_ANOMALY_MALICIOUS_OAUTH_APP_CONSENT` |
| **Nombre engañoso de aplicación de OAuth** | `ALERT_CABINET_APP_PERMISSION_ANOMALY_MISLEADING_APP_NAME` |
| **Nombre engañoso de publicador para una aplicación de OAuth** | `ALERT_CABINET_APP_PERMISSION_ANOMALY_MISLEADING_PUBLISHER_NAME` |
| **Se detectó una nueva aplicación** | `ALERT_CABINET_DISCOVERY_NEW_SERVICE` |
| **Una aplicación de OAuth usa una dirección URL de redireccionamiento no segura** | `ALERT_CABINET_APP_PERMISSION_ANOMALY_NON_SECURE_REDIRECT_URL` |
| **Alerta de directiva de aplicación de OAuth** | `ALERT_CABINET_APP_PERMISSION` |
| **Alerta de actividad sospechosa** | `ALERT_SUSPICIOUS_ACTIVITY` |
| **Alerta de uso sospechoso de la nube** | `ALERT_DISCOVERY_ANOMALY_DETECTION` |
| **Nombre de aplicación de OAuth sospechoso** | `ALERT_CABINET_APP_PERMISSION_ANOMALY_SUSPICIOUS_APP_NAME` |
| **Error de conector de aplicaciones de alerta del sistema** | `ALERT_MANAGEMENT_DISCONNECTED_API` |
| **Alerta del sistema: Error al cargar automáticamente los registros de Cloud Discovery** | `ALERT_MANAGEMENT_LOG_COLLECTOR_LOW_RATE` |
| **Alerta del sistema: Error al procesar los registros de Cloud Discovery** | `ALERT_MANAGEMENT_LOG_COLLECTOR_CONSTANTLY_FAILED_PARSING` |
| **Error del conector ICAP de alertas del sistema** | `ALERT_MANAGEMENT_DLP_CONNECTOR_ERROR` |
| **Error del agente SIEM de alertas del sistema** | `ALERT_MANAGEMENT_DISCONNECTED_SIEM` |
| **Notificaciones del agente SIEM de alertas del sistema** | `ALERT_MANAGEMENT_NOTIFICATIONS_SIEM` |
| **Región inusual para el recurso en la nube** | `MCAS_ALERT_ANUBIS_DETECTION_UNCOMMON_CLOUD_REGION` |
|

## <a name="next-steps"></a>Pasos siguientes

- [Conexión de Microsoft 365 Defender](connect-microsoft-365-defender.md) a Azure Sentinel.
- Obtenga más información sobre [Azure Sentinel](overview.md), [Microsoft 365 Defender](/microsoft-365/security/defender/microsoft-365-defender) y [Cloud App Security](/cloud-app-security/what-is-cloud-app-security).
