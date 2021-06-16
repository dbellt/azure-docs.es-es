---
title: Integración de Splunk mediante Azure Monitor | Microsoft Docs
description: Obtenga información sobre cómo integrar los registros de Azure Active Directory con Splunk mediante Azure Monitor.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 2c3db9a8-50fa-475a-97d8-f31082af6593
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 03/10/2021
ms.author: markvi
ms.reviewer: besiler
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3ff9a6c989446c2cee074485ba02fddd40cbfc83
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111953324"
---
# <a name="how-to-integrate-azure-active-directory-logs-with-splunk-using-azure-monitor"></a>Procedimientos: Integrar los registros de Azure Active Directory con Splunk mediante Azure Monitor

En este artículo, se ofrece información sobre cómo integrar los registros de Azure Active Directory (Azure AD) con Splunk mediante Azure Monitor. En primer lugar, se enrutan los registros a un centro de eventos de Azure y, a continuación, se integra el centro de eventos con Splunk.

## <a name="prerequisites"></a>Prerrequisitos

Para usar esta característica, necesita:

- Un centro de eventos de Azure que contenga registros de actividad de Azure AD. Aprenda cómo [transmitir los registros de actividad a un centro de eventos](./tutorial-azure-monitor-stream-logs-to-event-hub.md). 

-  El [complemento Splunk para Microsoft Cloud Services](https://splunkbase.splunk.com/app/3110/#/details). 

## <a name="integrate-azure-active-directory-logs"></a>Integración de registros de Azure Active Directory 

1. Abra la instancia de Splunk y seleccione **Data Summary** (Resumen de datos).

    ![Botón "Data Summary" (Resumen de datos)](./media/howto-integrate-activity-logs-with-splunk/DataSummary.png)

2. Seleccione la pestaña **Sourcetypes** (Tipos de origen) y, a continuación, seleccione **amal: aadal:audit**

    ![Pestaña Sourcetypes (Tipos de origen) del resumen de datos](./media/howto-integrate-activity-logs-with-splunk/sourcetypeaadal.png)

    Los registros de actividad de Azure AD se muestran en la ilustración siguiente:

    ![Registros de actividad](./media/howto-integrate-activity-logs-with-splunk/activitylogs.png)

> [!NOTE]
> Si no puede instalar un complemento en la instancia de Splunk (por ejemplo, si usa un proxy o se ejecuta en Splunk Cloud), puede reenviar estos eventos al recopilador de eventos de HTTP de Splunk. Para ello, use esta [función de Azure](https://github.com/Microsoft/AzureFunctionforSplunkVS) que se desencadena con los nuevos mensajes del centro de eventos. 
>

## <a name="next-steps"></a>Pasos siguientes

* [Interpretación del esquema de registros de auditoría en Azure Monitor](./overview-reports.md)
* [Interpretación del esquema de registros de inicio de sesión en Azure Monitor](reference-azure-monitor-sign-ins-log-schema.md)
* [Preguntas frecuentes y problemas conocidos](concept-activity-logs-azure-monitor.md#frequently-asked-questions)