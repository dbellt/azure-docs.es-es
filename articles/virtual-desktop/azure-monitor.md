---
title: Uso de Azure Monitor para Windows Virtual Desktop (Azure)
description: Uso de Azure Monitor para Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 03/31/2020
ms.author: helohr
manager: femila
ms.openlocfilehash: 69edeb73be969c1f065ff99ee80153305ec70a60
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109753712"
---
# <a name="use-azure-monitor-for-windows-virtual-desktop-to-monitor-your-deployment"></a>Uso de Azure Monitor para Windows Virtual Desktop para supervisar implementaciones

Azure Monitor para Windows Virtual Desktop es un panel basado en libros de Azure Monitor que ayuda a que los profesionales de TI entiendan sus entornos de Windows Virtual Desktop. Este tema lo guiará a través del proceso de configuración de Azure Monitor para Windows Virtual Desktop con el objetivo de supervisar los entornos de Windows Virtual Desktop.

## <a name="requirements"></a>Requisitos

Antes de empezar a usar Azure Monitor para Windows Virtual Desktop, debe configurar lo siguiente:

- Todos los entornos de Windows Virtual Desktops que supervise deben estar basados en la versión más reciente de Windows Virtual Desktop que sea compatible con Azure Resource Manager.
- Al menos un área de trabajo de Log Analytics configurada. Utilice un área de trabajo de Log Analytics designada para los hosts de sesión de Windows Virtual Desktop. De este modo, se asegurará de que los contadores de rendimiento y los eventos solo se recopilan de los hosts de sesión de su implementación de Windows Virtual Desktop.
- Habilite la recopilación de datos de los siguientes componentes del área de trabajo de Log Analytics:
    - Diagnósticos del entorno de Windows Virtual Desktop
    - Contadores de rendimiento recomendados de los hosts de sesión de Windows Virtual Desktop
    - Registros de eventos de Windows recomendados de los hosts de sesión Windows Virtual Desktop

 El proceso de configuración de datos que se describe en este artículo es el único que necesitará para supervisar Windows Virtual Desktop. Para ahorrar costos, puede deshabilitar todos los demás elementos que envían datos al área de trabajo de Log Analytics.

Cualquier persona que supervise Azure Monitor para Windows Virtual Desktop en su entorno necesitará también los siguientes permisos de acceso de lectura:

- Acceso de lectura a las suscripciones de Azure que contienen los recursos de Windows Virtual Desktop
- Acceso de lectura a los grupos de recursos de la suscripción que contienen los hosts de la sesión de Windows Virtual Desktop
- Acceso de lectura al área o áreas de trabajo de Log Analytics

>[!NOTE]
> El acceso de lectura solo permite que los administradores vean los datos. Necesitarán permisos diferentes para administrar los recursos en el portal de Windows Virtual Desktop.

## <a name="open-azure-monitor-for-windows-virtual-desktop"></a>Apertura de Azure Monitor para Windows Virtual Desktop

Puede abrir Azure Monitor para Windows Virtual Desktop con uno de los métodos siguientes:

- Vaya a [aka.ms/azmonwvdi](https://aka.ms/azmonwvdi).
- Busque y seleccione **Windows Virtual Desktop** en Azure Portal y, a continuación, seleccione **Conclusiones**.
- Busque y seleccione **Azure Monitor** en Azure Portal. Seleccione **Insights Hub** (Centro de conclusiones) en **Conclusiones** y haga clic en **Windows Virtual Desktop**.
Cuando se abra la página, especifique la **suscripción**, el **grupo de recursos**, el **grupo de hosts** y el **intervalo de tiempo** del entorno que desea supervisar.

>[!NOTE]
>Actualmente, Windows Virtual Desktop solo permite supervisar una suscripción, un grupo de recursos y un grupo de hosts al mismo tiempo. Si no puede encontrar el entorno que desea supervisar, consulte [nuestra documentación de solución de problemas](troubleshoot-azure-monitor.md) para ver las solicitudes de características y los problemas conocidos.

## <a name="log-analytics-settings"></a>Configuración de Log Analytics

Si desea utilizar Azure Monitor para Windows Virtual Desktop, necesitará al menos un área de trabajo de Log Analytics. Utilice un área de trabajo de Log Analytics designada para los hosts de sesión de Windows Virtual Desktop. De este modo, se asegurará de que los contadores de rendimiento y los eventos solo se recopilan de los hosts de sesión de su implementación de Windows Virtual Desktop. Si ya tiene un área de trabajo configurada, vaya a [Configuración con el libro](#set-up-using-the-configuration-workbook). Si necesita configurar una, consulte este artículo sobre la [creación de un área de trabajo de Log Analytics en Azure Portal](../azure-monitor/logs/quick-create-workspace.md).

>[!NOTE]
>Se aplicarán los cargos de almacenamiento estándar de Log Analytics. Para empezar, se recomienda elegir el modelo de pago por uso y ajustarlo a medida que escale la implementación y recopile más datos. Para obtener más información, consulte la sección [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

## <a name="set-up-using-the-configuration-workbook"></a>Configuración con el libro

Si esta es la primera vez que abre Azure Monitor para Windows Virtual Desktop, tendrá que configurar Azure Monitor para su entorno de Windows Virtual Desktop. Para configurar los recursos:

1. Abra Azure Monitor para Windows Virtual Desktop en Azure Portal, en [aka.ms/azmonwvdi](https://aka.ms/azmonwvdi), y seleccione **configuration workbook** (libro de configuración).
2. Seleccione el entorno que desee configurar en **Suscripción**, **Grupo de recursos** y **Grupo de host**.

El libro de configuración configura el entorno de supervisión y permite comprobar la configuración después de finalizar el proceso de configuración. Es importante que compruebe la configuración si los elementos del panel no se muestran correctamente o si el grupo de productos publica actualizaciones que requieren nuevas configuraciones.

### <a name="resource-diagnostic-settings"></a>Configuración de diagnóstico de recursos

Para recopilar información sobre la infraestructura de Windows Virtual Desktop, deberá habilitar algunas opciones de configuración de diagnóstico en las áreas de trabajo y los grupos de hosts de Windows Virtual Desktop (es decir, en el área de trabajo de Windows Virtual Desktop, no en la de Log Analytics). Para más información sobre los grupos de hosts, las áreas de trabajo y otros objetos de recursos de Windows Virtual Desktop, consulte nuestra [guía del entorno](environment-setup.md).

Encontrará más detalles sobre los diagnósticos de Windows Virtual Desktop y las tablas de diagnóstico compatibles en este artículo sobre el [envío de diagnósticos de Windows Virtual Desktop a Log Analytics](diagnostics-log-analytics.md).

Para configurar el diagnóstico de recursos en el libro de configuración: 

1. Seleccione la pestaña **Configuración de diagnóstico del recurso** en el libro de configuración. 
2. Seleccione el **área de trabajo de Log Analytics** a la que desea enviar los diagnósticos de Windows Virtual Desktop. 

#### <a name="host-pool-diagnostic-settings"></a>Configuración de diagnóstico del grupo de hosts

Para configurar los diagnósticos de un grupo de hosts utilizando la sección Configuración de diagnóstico del recurso del libro de configuración:

1. En **Grupos de host**, compruebe si los diagnósticos de Windows Virtual Desktop están habilitados. Si no es así, aparecerá este mensaje de error: "No existing diagnostic configuration was found for the selected host pool" (No se encontró ninguna configuración de diagnóstico para el grupo de hosts seleccionado). Deberá habilitar las siguientes tablas de diagnóstico compatibles:

    - Punto de control
    - Error
    - Administración
    - Conexión
    - HostRegistration
    - AgentHealthStatus
    
    >[!NOTE]
    > Si no ve el mensaje de error, no es necesario que realice los pasos 2 a 4.

2. Seleccione **Configure host pool** (Configurar grupo de hosts).
3. Seleccione **Implementar**.
4. Actualice el libro de configuración.

#### <a name="workspace-diagnostic-settings"></a>Configuración de diagnóstico del área de trabajo 

Para configurar los diagnósticos del área de trabajo utilizando la sección Configuración de diagnóstico del recurso del libro de configuración:

 1. En **Área de trabajo**, compruebe si los diagnósticos de Windows Virtual Desktop están habilitados para el área de trabajo de esta aplicación. Si no es así, aparecerá este mensaje de error: "No existing diagnostic configuration was found for the selected workspace." (No se encontró ninguna configuración de diagnóstico para el grupo de hosts seleccionado). Deberá habilitar las siguientes tablas de diagnóstico compatibles:
 
    - Punto de control
    - Error
    - Administración
    - Fuente
    
    >[!NOTE]
    > Si no ve el mensaje de error, no es necesario que realice los pasos 2 a 4.

2. Seleccione **Configurar área de trabajo**.
3. Seleccione **Implementar**.
4. Actualice el libro de configuración.

### <a name="session-host-data-settings"></a>Configuración de los datos de los hosts de sesión

Para recopilar información sobre los hosts de sesión de Windows Virtual Desktop, deberá instalar el agente de Log Analytics en todos los hosts de sesión del grupo, asegurarse de que envían información a un área de trabajo de Log Analytics y configurar los valores del agente de Log Analytics para recopilar los datos de rendimiento y los registros de eventos de Windows.

El área de trabajo de Log Analytics a la que se envían los datos de los hosts de sesión no tiene que ser la misma que el área de trabajo a la que se envían los datos de diagnóstico. Si tiene hosts de sesión de Azure fuera del entorno de Windows Virtual Desktop, es conveniente que tenga un área de trabajo de Log Analytics designada para los hosts de sesión de Windows Virtual Desktop. 

Para establecer el área de trabajo de Log Analytics en la que desea recopilar los datos de los hosts de sesión: 

1. Seleccione la pestaña **Session host data settings** (Configuración de datos de hosts de sesión) en el libro de configuración. 
2. Seleccione el **área de trabajo de Log Analytics** a la que desea enviar los datos de los hosts de sesión. 

#### <a name="session-hosts"></a>Hosts de sesión

Por último, deberá instalar el agente de Log Analytics en todos los hosts de sesión del grupo y enviar sus datos al área de trabajo de Log Analytics seleccionada. Si Log Analytics no está configurado para todos los hosts de sesión del grupo, en la sección **Session hosts** (Hosts de sesión) de la parte superior de **Session host data settings** (Configuración de datos de hosts de sesión), aparecerá el mensaje "Some hosts in the host pool are not sending data to the selected Log Analytics workspace" (Algunos hosts del grupo no están enviando datos al área de trabajo de Log Analytics seleccionada).

>[!NOTE]
> Si no ve la sección **Session hosts** (Hosts de sesión) ni el mensaje de error, significa que todos los hosts de sesión están configurados correctamente. En ese caso, continúe con las instrucciones de configuración de [Contadores de rendimiento del área de trabajo](#workspace-performance-counters).

Para configurar los hosts de sesión restantes utilizando el libro de configuración:

1. Seleccione **Add hosts to workspace** (Agregar hosts al área de trabajo). 
2. Actualice el libro de configuración.

>[!NOTE]
>La máquina host debe estar en ejecución para instalar la extensión de Log Analytics. Si se produce un error en la implementación automática, puede instalar manualmente la extensión en un host. Para obtener información sobre cómo instalar la extensión manualmente, consulte [Extensión de máquina virtual de Log Analytics para Windows](../virtual-machines/extensions/oms-windows.md).

#### <a name="workspace-performance-counters"></a>Contadores de rendimiento del área de trabajo

Tendrá que habilitar contadores de rendimiento específicos para recopilar información de rendimiento de los hosts de sesión y enviarla al área de trabajo de Log Analytics.

Si ya tiene contadores de rendimiento habilitados y desea quitarlos, siga las instrucciones que se indican en este artículo sobre la [configuración de contadores de rendimiento](../azure-monitor/agents/data-sources-performance-counters.md). Puede agregar y quitar a la vez contadores de rendimiento en la misma ubicación.

Para configurar los contadores de rendimiento utilizando el libro de configuración: 

1. En la sección **Workspace performance counters** (Contadores de rendimiento del área de trabajo) del libro de configuración, active **Configured counters** (Contadores configurados) para ver los contadores que ya están habilitados y pueden enviar datos al área de trabajo de Log Analytics. Compruebe los **Missing counters** (Contadores ausentes) para asegurarse de que ha habilitado todos los contadores necesarios.
2. Si hay contadores que faltan, seleccione **Configurar contadores de rendimiento**.
3. Seleccione **Aplicar configuración**.
4. Actualice el libro de configuración.
5. Asegúrese de que todos los contadores necesarios estén habilitados; para ello, compruebe la lista de **Missing counters** (Contadores ausentes). 

#### <a name="configure-windows-event-logs"></a>Configuración de registros de eventos de Windows

También deberá habilitar registros de eventos de Windows específicos para recopilar errores, advertencias e información de los hosts de sesión y enviarlos al área de trabajo de Log Analytics.

Si ya ha habilitado los registros de eventos de Windows y desea quitarlos, siga las instrucciones que se indican en este artículo sobre la [configuración de registros de eventos de Windows](../azure-monitor/agents/data-sources-windows-events.md#configuring-windows-event-logs).  Puede agregar y quitar a la vez registros de eventos de Windows en la misma ubicación.

Para configurar los registros de eventos de Windows utilizando el libro de configuración:

1. En **Windows Event Logs configuration** (Configuración de registros de eventos de Windows), active **Configured Event Logs** (Registros de eventos configurados) para ver los registros de eventos que ya ha habilitado para que se envíen al área de trabajo de Log Analytics. Consulte **Missing Event Logs** (Registros de eventos ausentes) y compruebe que se han habilitado todos los registros de eventos de Windows.
2. Si faltan registros de eventos de Windows, seleccione **Configurar eventos**.
3. Seleccione **Implementar**.
4. Actualice el libro de configuración.
5. Asegúrese de que todos los registros de eventos de Windows necesarios estén habilitados; para ello, compruebe la lista de **Missing Event Logs** (Registros de eventos ausentes). 

>[!NOTE]
>Si se produce un error en la implementación automática de eventos, seleccione **Open agent configuration** (Abrir configuración del agente) en el libro de configuración para agregar manualmente los registros de eventos de Windows que falten. 

## <a name="optional-configure-alerts"></a>Opcional: configuración de MPIO

Azure Monitor para Windows Virtual Desktop permite supervisar las alertas de Azure Monitor que se generan en la suscripción seleccionada en relación con los datos de Windows Virtual Desktop. Las alertas de Azure Monitor son una característica opcional de las suscripciones de Azure y deben configurarse en otro proceso distinto al de Azure Monitor para Windows Virtual Desktop. Puede usar el marco de alertas de Azure Monitor para configurar alertas personalizadas sobre los eventos, diagnósticos y recursos de Windows Virtual Desktop. Para más información sobre las alertas de Azure Monitor, consulte [Alertas de registro a Azure Monitor](../azure-monitor/alerts/alerts-log.md).

## <a name="diagnostic-and-usage-data"></a>Datos de diagnóstico y uso

Microsoft recopila automáticamente datos de uso y rendimiento a través del servicio Azure Monitor. Microsoft usa estos datos para mejorar la calidad, la seguridad y la integridad del servicio.

Para proporcionar funcionalidades de solución de problemas precisas y eficaces, los datos recopilados incluyen el identificador de sesión del portal, el identificador de usuario de Azure Active Directory y el nombre de la pestaña del portal donde se produjo el evento. Microsoft no recopila nombres, direcciones ni otra información de contacto.

Para más información sobre el uso y la recopilación de datos, vea la [Declaración de privacidad de Microsoft Online Services](https://privacy.microsoft.com/privacystatement).

>[!NOTE]
>Para obtener información sobre cómo ver o eliminar los datos personales recopilados por el servicio, vea [Solicitudes de titulares de los datos de Azure para el RGPD y la CCPA](/microsoft-365/compliance/gdpr-dsr-azure). Si quiere conocer más detalles del Reglamento general de protección de datos, consulte la [sección sobre RGPD del Portal de confianza de servicios](https://servicetrust.microsoft.com/ViewPage/GDPRGetStarted).

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha configurado Azure Monitor para el entorno de Windows Virtual Desktop, le dejamos algunos recursos que podrían ayudarle a supervisar el entorno:

- Consulte nuestro [glosario](azure-monitor-glossary.md) para obtener más información sobre los términos y conceptos relacionados con Azure Monitor para Windows Virtual Desktop.
- Para calcular, medir y administrar los costos de almacenamiento de datos, consulte este artículo sobre el [cálculo de costos en Azure Monitor](azure-monitor-costs.md).
- Si encuentra algún problema, consulte nuestra [guía de solución de problemas](troubleshoot-azure-monitor.md) para obtener ayuda y ver los problemas conocidos.
