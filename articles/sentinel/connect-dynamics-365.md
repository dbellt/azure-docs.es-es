---
title: Conexión de registros de Dynamics 365 a Azure Sentinel | Microsoft Docs
description: Aprenda a usar el conector de actividades de Dynamics 365 Common Data Service (CDS) para proporcionar información sobre las actividades continuadas de administración, usuario y soporte técnico.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/03/2021
ms.author: yelevin
ms.openlocfilehash: 7d2a9c69108211b1342c553731c28785e1f2cc79
ms.sourcegitcommit: ef950cf37f65ea7a0f583e246cfbf13f1913eb12
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111421803"
---
# <a name="connect-dynamics-365-activity-logs-to-azure-sentinel"></a>Conexión de los registros de actividad de Dynamics 365 a Azure Sentinel

El conector de actividades de [Dynamics 365](/office365/servicedescriptions/microsoft-dynamics-365-online-service-description) Common Data Service (CDS) proporciona información sobre las actividades de administración, usuario y soporte técnico, así como eventos de registro de Microsoft Social Engagement. Si conecta los registros de Dynamics 365 CRM a Azure Sentinel, puede ver estos datos en libros, usarlos para crear alertas personalizadas y aprovecharlos para mejorar el proceso de investigación. Este nuevo conector de Azure Sentinel recopila los datos de Dynamics CDS de la API de Administración de Office. Para obtener más información acerca de las actividades de Dynamics CDS auditadas en Power Platform, visite [Habilitación y uso del registro de actividad](/power-platform/admin/enable-use-comprehensive-auditing).

[!INCLUDE [reference-to-feature-availability](includes/reference-to-feature-availability.md)]

## <a name="prerequisites"></a>Requisitos previos

- Debe tener permisos de lectura y escritura en el área de trabajo de Azure Sentinel.

- Debe tener una [licencia de producción de Microsoft Dynamics 365](/office365/servicedescriptions/microsoft-dynamics-365-online-service-description). Este conector no está disponible para los entornos de espacio aislado.
    - Se requiere una suscripción a Microsoft 365 Enterprise [E3 o E5](/power-platform/admin/enable-use-comprehensive-auditing#requirements) para realizar el registro de actividad.

- Para extraer datos de la API de Administración de Office:
    - Debe ser administrador global en su inquilino.

    - El [registro de auditoría de Office](/office365/servicedescriptions/office-365-platform-service-description/office-365-securitycompliance-center) debe estar habilitado en el [Centro de seguridad y cumplimiento de Office](/microsoft-365/compliance/search-the-audit-log-in-security-and-compliance).

## <a name="enable-the-dynamics-365-activities-data-connector"></a>Habilitación del conector de datos de actividades de Dynamics 365

1. En el menú de navegación de Azure Sentinel, seleccione **Conectores de datos**.

1. En la galería de **Conectores de datos**, seleccione **Dynamics 365** y, a continuación, seleccione **Open connector page** (Abrir página de conectores) en el panel de vista previa.

1. En la pestaña **Instrucciones**, en **Configuración**, haga clic en **Conectar**. 

    Una vez activado el conector, tardará unos 30 minutos en poder ver los datos que se incorporan al gráfico. 

    Según el [registro de auditoría de Office en el Centro de cumplimiento](/microsoft-365/compliance/search-the-audit-log-in-security-and-compliance#requirements-to-search-the-audit-log), puede tardar hasta 30 minutos o 24 horas después de que se produzca un evento para que se devuelva el registro de auditoría correspondiente en los resultados.

1. Los registros de auditoría de Microsoft Dynamics se pueden encontrar en la tabla `Dynamics365Activity`. Consulte la [referencia de esquema](/azure/azure-monitor/reference/tables/dynamics365activity) de la tabla.

## <a name="querying-the-data"></a>Consultas de datos

1. En el menú de navegación de Azure Sentinel, seleccione **Registros**.

1. Ejecute la siguiente consulta para comprobar que llegan los registros:

    ```kusto
    Dynamics365Activity
    | take 10
    ```


## <a name="next-steps"></a>Pasos siguientes
En este documento, ha aprendido a conectar los datos de actividades de Dynamics 365 a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a detectar amenazas con Azure Sentinel mediante las reglas [integradas](tutorial-detect-threats-built-in.md) o [personalizadas](tutorial-detect-threats-custom.md).
