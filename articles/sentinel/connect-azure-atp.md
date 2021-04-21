---
title: Conexión de los datos de Microsoft Defender for Identity (antes Azure ATP) con Azure Sentinel | Microsoft Docs
description: Aprenda a transmitir registros de Microsoft Defender for Identity (antes Azure Advanced Threat Protection [ATP]) a Azure Sentinel con un solo clic.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/30/2019
ms.author: yelevin
ms.openlocfilehash: 7a26091d4985b6fdb17120c6fd70476a750c94a9
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107714122"
---
# <a name="connect-data-from-microsoft-defender-for-identity-formerly-azure-advanced-threat-protection"></a>Conexión de datos de Microsoft Defender for Identity (antes Azure Advanced Threat Protection)

> [!IMPORTANT]
> El conector de datos Microsoft Defender for Identity se encuentra actualmente en versión preliminar pública.
> Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este artículo se describe cómo transmitir alertas de seguridad desde [Microsoft Defender for Identity](/azure-advanced-threat-protection/what-is-atp) a Azure Sentinel. 

Para reenviar alertas de mantenimiento, además de las de seguridad, integre Microsoft Defender for Identity con un servidor de Syslog. Para obtener más información, vea la [documentación de Microsoft Defender for Identity](/defender-for-identity/setting-syslog). 

## <a name="prerequisites"></a>Requisitos previos

- Un usuario que sea administrador global o que tenga permisos de administrador de seguridad
- Debe ser un cliente de la versión preliminar de Microsoft Defender for Identity y habilitar la integración entre Microsoft Defender for Identity y Microsoft Cloud App Security. Para más información, consulte [Integración de Microsoft Defender for Identity](/cloud-app-security/mdi-integration).

## <a name="connect-to-microsoft-defender-for-identity"></a>Conexión con Microsoft Defender for Identity

Asegúrese de que la versión preliminar de Microsoft Defender for Identity esté [habilitada en la red](/azure-advanced-threat-protection/install-atp-step1).
Si Microsoft Defender for Identity está implementado e ingiriendo sus datos, las alertas de actividades sospechosas se pueden transmitir muy fácilmente a Azure Sentinel. Las alertas pueden tardar hasta 24 horas en empezar a transmitirse a Azure Sentinel.


1. Para conectar Microsoft Defender for Identity con Azure Sentinel, primero debe habilitar la integración entre Microsoft Defender for Identity y Microsoft Cloud App Security. Para información sobre cómo hacerlo, consulte [Integración de Microsoft Defender for Identity](/cloud-app-security/mdi-integration).

1. En Azure Sentinel, seleccione **Data Connectors** (Conectores de datos) y, luego, haga clic en el icono **Microsoft Defender for Identity (Preview)** (Microsoft Defender for Identity [versión preliminar]).

1. Puede seleccionar si quiere que las alertas de Microsoft Defender for Identity generen incidentes automáticamente en Azure Sentinel. En **Creación de incidentes** seleccione **Habilitar** para habilitar la regla analítica predeterminada que crea incidentes automáticamente a partir de alertas generadas en el servicio de seguridad conectado. Luego, puede editar esta regla en **Analytics** (Análisis) y **Active rules** (Reglas activas).

1. Haga clic en **Conectar**.

1. Para usar el esquema correspondiente en Log Analytics con las alertas de Microsoft Defender for Identity, busque **SecurityAlert**.

> [!NOTE]
> Si las alertas superan los 30 KB, Azure Sentinel dejará de mostrar el campo Entidades en las alertas.

## <a name="next-steps"></a>Pasos siguientes
En este documento, ha aprendido a conectar Microsoft Defender for Identity con Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).
