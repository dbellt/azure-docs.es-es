---
title: Transmisión de registros de Azure Active Directory a registros de Azure Monitor | Microsoft Docs
description: Aprenda a integrar registros de Azure Active Directory con registros de Azure Monitor.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.assetid: 2c3db9a8-50fa-475a-97d8-f31082af6593
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 06/11/2021
ms.author: markvi
ms.reviewer: besiler
ms.collection: M365-identity-device-management
ms.openlocfilehash: c29b631d3002f0c79fb2dd1b1f26dfa65051368c
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112018982"
---
# <a name="integrate-azure-ad-logs-with-azure-monitor-logs"></a>Integración de registros de Azure AD con registros de Azure Monitor

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

Registros de Azure Monitor permite consultar los datos para buscar eventos concretos, analizar tendencias y realizar la correlación entre varios orígenes de datos. Con la integración de los registros de actividad de Azure AD en registros de Azure Monitor, ahora puede realizar tareas como:

 * Comparar los registros de inicio de sesión de Azure AD con los registros de seguridad publicados por Azure Security Center.

 * Solucionar problemas de cuellos de botella de rendimiento en la página de inicio de sesión de la aplicación mediante la correlación de datos de rendimiento de la aplicación de Azure Application Insights.  

El siguiente vídeo es de una sesión de Ignite y muestra las ventajas del uso de registros de Azure Monitor para registros de Azure AD en escenarios de usuario prácticos.

> [!VIDEO https://www.youtube.com/embed/MP5IaCTwkQg?start=1894]

En este artículo, se aprende a integrar los registros de Azure Active Directory (Azure AD) con Azure Monitor.

## <a name="supported-reports"></a>Informes admitidos

Puede enrutar los registros de actividad de auditoría y los registros de actividad de inicio de sesión en registros de Azure Monitor para su posterior análisis. 

* **Registros de auditoría**: el [informe de actividad de registros de auditoría](concept-audit-logs.md) le proporciona acceso al historial de todas las tareas llevadas a cabo en el inquilino.
* **Registros de inicio de sesión**: Con el [informe de actividad de inicios de sesión](concept-sign-ins.md), puede determinar quién ha realizado las tareas notificadas en el informe de registros de auditoría.
* **Registros de aprovisionamiento**: Con los [registros de aprovisionamiento](../app-provisioning/application-provisioning-log-analytics.md), puede supervisar qué usuarios se han creado, actualizado y eliminado en todas las aplicaciones de terceros. 

> [!NOTE]
> En este momento no se admiten los registros de actividad de auditoría e inicio de sesión relacionados con B2C.
>

## <a name="prerequisites"></a>Prerrequisitos 

Para usar esta característica, necesita:

* Suscripción a Azure. Si no tiene ninguna suscripción de Azure, puede [registrarse para obtener una evaluación gratuita](https://azure.microsoft.com/free/).
* Un inquilino de Azure AD.
* Un usuario que sea *administrador global* o *administrador de seguridad* para el inquilino de Azure AD.
* Un área de trabajo de Log Analytics en la suscripción a Azure. Aprenda a [crear un área de trabajo de Log Analytics](../../azure-monitor/logs/quick-create-workspace.md).

## <a name="licensing-requirements"></a>Requisitos de concesión de licencia

Necesita un inquilino de Azure AD Premium P1 o P2 para usar esta característica. Puede encontrar el tipo de licencia del inquilino en la página **[Información general](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview)** en **Azure Active Directory**.

![Información del inquilino](./media/howto-integrate-activity-logs-with-log-analytics/tenant-information.png)
 
Si desea saber durante cuánto tiempo se almacenan los datos de actividad en un inquilino prémium, consulte [¿Durante cuánto tiempo Azure AD almacena los datos?](reference-reports-data-retention.md#how-long-does-azure-ad-store-the-data)

## <a name="send-logs-to-azure-monitor"></a>Envío de registros a Azure Monitor

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 

2. Seleccione **Azure Active Directory** > **Configuración de diagnóstico** -> **Agregar configuración de diagnóstico**. También puede seleccionar **Exportar configuraciones** desde la página **Registros de auditoría** o **Inicios de sesión** para ir a la página de configuración de diagnóstico.  
    
3. En el menú **Configuración de diagnóstico**, seleccione la casilla **Enviar a área de trabajo de Log Analytics** y, después, seleccione **Configurar**.

4. Seleccione el área de trabajo de Log Analytics a la que quiere enviar los registros o cree una nueva área de trabajo en el cuadro de diálogo proporcionado.  

5. Realice alguna de las siguientes acciones o ambas:
    * Para enviar los registros de auditoría al área de trabajo de Log Analytics, seleccione la casilla **AuditLogs**. 
    * Para enviar los registros de inicio de sesión al área de trabajo de Log Analytics, seleccione la casilla **SignInLogs**.

6. Seleccione **Guardar** para guardar la configuración.

    ![Configuración de diagnóstico](./media/howto-integrate-activity-logs-with-log-analytics/Configure.png)

7. Después de unos 15 minutos, compruebe que los eventos se transmiten al área de trabajo de Log Analytics.

## <a name="next-steps"></a>Pasos siguientes

* [Análisis de registros de actividad de Azure AD con registros de Azure Monitor](howto-analyze-activity-logs-log-analytics.md)
* [Instalación y uso de las vistas de Log Analytics para Azure Active Directory](howto-install-use-log-analytics-views.md)