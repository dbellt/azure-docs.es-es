---
title: Supervisión de Azure AD B2C con Azure Monitor
titleSuffix: Azure AD B2C
description: Obtenga información acerca de cómo registrar eventos de Azure AD B2C con Azure Monitor mediante la administración de recursos delegados.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.author: mimart
ms.subservice: B2C
ms.date: 01/29/2021
ms.openlocfilehash: 4ac0cc618ec03d844c73961dcdb66f7357ce60f2
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109783790"
---
# <a name="monitor-azure-ad-b2c-with-azure-monitor"></a>Supervisión de Azure AD B2C con Azure Monitor

Use Azure Monitor para enrutar los registros de inicio de sesión y [auditoría](view-audit-logs.md) de Azure Active Directory B2C (Azure AD B2C) a distintas soluciones de supervisión. Puede conservar los registros para su uso a largo plazo o integrarlos en herramientas de Administración de eventos e información de seguridad (SIEM) de terceros para sacar conclusiones sobre su entorno.

Puede enrutar los eventos de registro a:

* Una [cuenta de Azure Storage](../storage/blobs/storage-blobs-introduction.md).
* Un [área de trabajo de Log Analytics](../azure-monitor/essentials/resource-logs.md#send-to-log-analytics-workspace) (para analizar los datos, crear paneles y alertar sobre eventos específicos).
* Un [centro de eventos](../event-hubs/event-hubs-about.md) de Azure (e integrarlo en sus instancias de Splunk y Sumo Logic).

![Azure Monitor](./media/azure-monitor/azure-monitor-flow.png)

En este artículo, aprenderá a transferir los registros a un área de trabajo de Azure Log Analytics. Después, puede crear un panel o crear alertas basadas en las actividades de los usuarios en Azure AD B2C.

> [!IMPORTANT]
> Al planear la transferencia de registros de Azure AD B2C a diferentes soluciones de supervisión, o repositorio, tenga en cuenta lo siguiente. Los registros de Azure AD B2C contienen datos personales. Estos datos se deben procesar de una manera que garantice la seguridad adecuada de los datos personales, incluida la protección frente al procesamiento no autorizado o ilícito, utilizando las medidas técnicas u organizativas adecuadas.


## <a name="deployment-overview"></a>Introducción a la implementación

Azure AD B2C aprovecha la [supervisión de Azure Active Directory](../active-directory/reports-monitoring/overview-monitoring.md). Para habilitar la *configuración de diagnóstico* en Azure Active Directory dentro de su inquilino de Azure AD B2C, use [Azure Lighthouse](../lighthouse/overview.md) para [delegar un recurso](../lighthouse/concepts/architecture.md), lo que permite que su instancia de Azure AD B2C (el **proveedor de servicios**) administre un recurso de Azure AD (el **cliente**). Después de completar los pasos de este artículo, tendrá acceso al grupo de recursos *azure-ad-b2c-monitor* que contiene el [área de trabajo Log Analytics](../azure-monitor/logs/quick-create-workspace.md) en el portal de **Azure AD B2C**. También podrá transferir los registros desde Azure AD B2C al área de trabajo de Log Analytics.

Durante esta implementación, autorizará a un usuario o grupo en el directorio de Azure AD B2C para configurar la instancia del área de trabajo de Log Analytics en el inquilino que contiene su suscripción a Azure. Para crear la autorización, implemente una plantilla de [Azure Resource Manager](../azure-resource-manager/index.yml) en el inquilino de Azure AD que contiene la suscripción.

En el siguiente diagrama se muestran los componentes que se configurarán en los inquilinos de Azure AD y Azure AD B2C.

![Proyección del grupo de recursos](./media/azure-monitor/resource-group-projection.png)

Durante esta implementación, configurará el inquilino de Azure AD B2C y el de Azure AD donde se hospedará el área de trabajo de Log Analytics. Se debe asignar el rol [Administrador global](../active-directory/roles/permissions-reference.md#global-administrator) a la cuenta de Azure AD B2C en el inquilino de Azure AD B2C. La cuenta de Azure AD utilizada para ejecutar la implementación debe tener asignado el rol [Propietario](../role-based-access-control/built-in-roles.md#owner) en la suscripción de Azure AD. También es importante asegurarse de haber iniciado sesión en el directorio correcto a medida que completa cada paso según lo descrito.

## <a name="1-create-or-choose-resource-group"></a>1. Crear o elegir un grupo de recursos

En primer lugar, cree o elija un grupo de recursos que contenga el área de trabajo de Log Analytics de destino que recibirá los datos de Azure AD B2C. El nombre del grupo de recursos se especificará al implementar la plantilla de Azure Resource Manager.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el **inquilino de Azure AD**.
1. [Cree un grupo de recursos](../azure-resource-manager/management/manage-resource-groups-portal.md#create-resource-groups) o elija uno existente. En este ejemplo, se usa un grupo de recursos denominado *azure-ad-b2c-monitor*.

## <a name="2-create-a-log-analytics-workspace"></a>2. Creación de un área de trabajo de Log Analytics

Un **área de trabajo de Log Analytics** es un entorno único de datos de registro de Azure Monitor. Esta área de trabajo de Log Analytics se usará para recopilar datos de los [registros de auditoría](view-audit-logs.md) de Azure AD B2C y, a continuación, visualizarlos con consultas y libros o crear alertas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el **inquilino de Azure AD**.
1. [Crear un área de trabajo de Log Analytics](../azure-monitor/logs/quick-create-workspace.md). En este ejemplo se usa un área de trabajo de Log Analytics denominada *AzureAdB2C*, en un grupo de recursos denominado *azure-ad-b2c-monitor*.

## <a name="3-delegate-resource-management"></a>3. Administración de recursos delegados

En este paso, se elige el inquilino de Azure AD B2C como **proveedor de servicios**. También puede definir las autorizaciones que necesite para asignar los roles integrados de Azure adecuados a grupos de su inquilino de Azure AD.

### <a name="31-get-your-azure-ad-b2c-tenant-id"></a>3.1 Obtención del identificador de inquilino de Azure AD B2C

En primer lugar, obtenga el **identificador de inquilino** del directorio de Azure AD B2C (también conocido como identificador de directorio).

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de **Azure AD B2C**.
1. Seleccione **Azure Active Directory** y luego **Información general**.
1. Anote el **Identificador de inquilino**.

### <a name="32-select-a-security-group"></a>3.2 Selección de un grupo de seguridad

Seleccione ahora un grupo o usuario de Azure AD B2C al que desee conceder permiso para el grupo de recursos que creó anteriormente en el directorio que contiene la suscripción.  

Para facilitar la administración, se recomienda usar *grupos* de usuarios de Azure AD para cada rol, lo que le permite agregar o quitar usuarios individuales en el grupo, en lugar de asignar permisos directamente a ese usuario. En este tutorial, se agregará un grupo de seguridad.

> [!IMPORTANT]
> Para agregar permisos a un grupo de Azure AD, el **Tipo de grupo** debe establecerse en **Seguridad**. Esta opción se selecciona cuando se crea el grupo. Para obtener más información vea [Creación de un grupo básico e incorporación de miembros con Azure Active Directory](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

1. Con **Azure Active Directory** todavía seleccionado en el directorio de **Azure AD B2C**, seleccione **Grupos** y, a continuación, seleccione un grupo. Si no tiene un grupo existente, cree un grupo de **seguridad** y, a continuación, agregue miembros. Para más información, siga el procedimiento [Creación de un grupo básico e incorporación de miembros con Azure Active Directory](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md). 
1. Seleccione **información general** y anote el **identificador de objeto** del grupo.

### <a name="33-create-an-azure-resource-manager-template"></a>3.3. Creación de una plantilla de Azure Resource Manager

A continuación, creará una plantilla de Azure Resource Manager que conceda a Azure AD B2C acceso al grupo de recursos de Azure AD que creó anteriormente (por ejemplo, *azure-ad-b2c-monitor*). Implemente la plantilla desde el ejemplo de GitHub mediante el botón **Deploy to Azure** (Implementar en Azure), que abre Azure Portal y le permite configurar e implementar la plantilla directamente en el portal. Para estos pasos, asegúrese de que ha iniciado sesión en su inquilino de Azure AD (no en el inquilino de Azure AD B2C).

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de **Azure AD**.
3. Use el botón **Deploy to Azure** (Implementar en Azure) para abrir Azure Portal e implementar la plantilla directamente en el portal. Para más información, consulte [Creación de una plantilla de Azure Resource Manager](../lighthouse/how-to/onboard-customer.md#create-an-azure-resource-manager-template).

   [![Implementación en Azure](https://aka.ms/deploytoazurebutton)](   https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fazure-ad-b2c%2Fsiem%2Fmaster%2Ftemplates%2FrgDelegatedResourceManagement.json)

5. En la página **Implementación personalizada**, especifique la siguiente información:

   | Campo   | Definición |
   |---------|------------|
   | Suscripción |  Seleccione el directorio que contiene la suscripción de Azure en la que se creó el grupo de recursos *azure-ad-b2c-monitor*. |
   | Region| Elija la región donde se va a implementar el recurso.  | 
   | Nombre de la oferta de MSP| Nombre que describe esta definición. Por ejemplo, *Supervisión de Azure AD B2C*.  |
   | Descripción de la oferta de MSP| Descripción breve de la oferta. Por ejemplo, *Enables Azure Monitor in Azure AD B2C*.|
   | Administrado por identificador de inquilino| El **identificador de inquilino** del inquilino de Azure AD B2C (también conocido como identificador de directorio). |
   |Autorizaciones|Especifique una matriz JSON de objetos que incluya `principalId`, `principalIdDisplayName` de Azure AD y `roleDefinitionId`de Azure. `principalId` es el **identificador de objeto** del grupo o usuario de B2C que tendrá acceso a los recursos de esta suscripción a Azure. Para este tutorial, especifique el identificador de objeto del grupo que anotó anteriormente. Para `roleDefinitionId`, use el valor [rol integrado](../role-based-access-control/built-in-roles.md) para el *rol Colaborador*, `b24988ac-6180-42a0-ab88-20f7382dd24c`.|
   | Nombre del grupo de recursos | Nombre del grupo de recursos que creó anteriormente en el inquilino de Azure AD. Por ejemplo, *azure-ad-b2c-monitor*. |

   En el ejemplo siguiente se muestra una matriz de autorizaciones con un grupo de seguridad.

   ```json
   [
       {
           "principalId": "<Replace with group's OBJECT ID>",
           "principalIdDisplayName": "Azure AD B2C tenant administrators",
           "roleDefinitionId": "b24988ac-6180-42a0-ab88-20f7382dd24c"
       }
   ]
   ```

Después de implementar la plantilla, la proyección de recursos puede tardar unos minutos en completarse (habitualmente menos de 5 minutos). Puede comprobar la implementación en el inquilino de Azure AD y obtener los detalles de la proyección de recursos. Para obtener más información, consulte [Visualización y administración de proveedores de servicios](../lighthouse/how-to/view-manage-service-providers.md).

## <a name="4-select-your-subscription"></a>4. Seleccione su suscripción.

Una vez que haya implementado la plantilla y esperado unos minutos a que se complete la proyección de recursos, siga estos pasos para asociar la suscripción al directorio de Azure AD B2C.

1. Cierre la sesión de Azure Portal si está iniciada actualmente (esto permite actualizar las credenciales de la sesión en el paso siguiente).
2. Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta de administrador de **Azure AD B2C**. Esta cuenta debe ser miembro del grupo de seguridad que especificó en el paso [Administración de recursos delegados](#3-delegate-resource-management).
3. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal.
4. Seleccione el directorio de Azure AD que contiene la suscripción de Azure y el grupo de recursos *azure-ad-b2c-monitor* que creó.

    ![Cambiar de directorio](./media/azure-monitor/azure-monitor-portal-03-select-subscription.png)

1. Verifique haber seleccionado el directorio y la suscripción correctos. En este ejemplo, todos los directorios y suscripciones están seleccionados.

    ![Todos los directorios seleccionados en el filtro Directorio y suscripción](./media/azure-monitor/azure-monitor-portal-04-subscriptions-selected.png)

## <a name="5-configure-diagnostic-settings"></a>5. Configuración de valores de diagnóstico

La configuración de diagnóstico define dónde se deben enviar los registros y las métricas de un recurso. Los posibles destinos son:

- [Cuenta de Almacenamiento de Azure](../azure-monitor/essentials/resource-logs.md#send-to-azure-storage)
- Soluciones de [Event Hubs](../azure-monitor/essentials/resource-logs.md#send-to-azure-event-hubs)
- [Área de trabajo de Log Analytics](../azure-monitor/essentials/resource-logs.md#send-to-log-analytics-workspace)

En este ejemplo, se usa el área de trabajo de Log Analytics para crear un panel.

### <a name="51-create-diagnostic-settings"></a>5.1 Creación de configuración de diagnóstico

Ya está listo para [crear configuraciones de diagnóstico](../active-directory/reports-monitoring/overview-monitoring.md) en Azure Portal.

Para establecer la configuración de supervisión de los registros de actividad de Azure AD B2C:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con su cuenta de administrador de Azure AD B2C. Esta cuenta debe ser miembro del grupo de seguridad que especificó en el paso [Selección de un grupo de seguridad](#32-select-a-security-group).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. Seleccione **Azure Active Directory**.
1. En **Supervisión**, seleccione **Configuración de diagnóstico**.
1. Si hay una configuración actual para el recurso, verá una lista de opciones ya configuradas. Puede seleccionar **Agregar configuración de diagnóstico** para agregar una nueva configuración o **Editar** para modificar una existente. Cada configuración no puede tener más de uno de los tipos de destino.

    ![Panel de configuración de diagnóstico en Azure Portal](./media/azure-monitor/azure-monitor-portal-05-diagnostic-settings-pane-enabled.png)

1. Asigne un nombre a la configuración, si aún no lo tiene.
1. Marque la casilla de cada destino para enviar los registros. Seleccione **Configurar** para especificar la configuración **como se describe en la tabla siguiente**.
1. Seleccione **Enviar a Log Analytics** y, a continuación, el **nombre del área de trabajo** que ha creado anteriormente (`AzureAdB2C`).
1. Seleccione **AuditLogs** y **SignInLogs**.
1. Seleccione **Guardar**.

> [!NOTE]
> Pueden transcurrir hasta 15 minutos para que un evento emitido [aparezca en un área de trabajo de Log Analytics](../azure-monitor/logs/data-ingestion-time.md). Obtenga más información acerca de las [latencias de informes de Active Directory](../active-directory/reports-monitoring/reference-reports-latencies.md), que pueden afectar a la obsolescencia de los datos y desempeñar un papel importante en la generación de informes.

Si ve el mensaje de error "Para configurar los valores de diagnóstico para usar Azure Monitor para el directorio de Azure AD B2C, debe configurar la administración de recursos delegada", asegúrese de iniciar sesión con un usuario que sea miembro del [grupo de seguridad](#32-select-a-security-group) y [seleccione su suscripción](#4-select-your-subscription).

## <a name="6-visualize-your-data"></a>6. Visualización de los datos

Ahora puede configurar el área de trabajo de Log Analytics para visualizar los datos y configurar alertas. Estas configuraciones se pueden realizar tanto en el inquilino de Azure AD como en el inquilino de Azure AD B2C.

### <a name="61-create-a-query"></a>6.1 Creación de una consulta

Las consultas de registro ayudan a aprovechar al máximo el valor de los datos recopilados en registros de Azure Monitor. Un lenguaje de consulta eficaz permite combinar datos de varias tablas, agregar grandes conjuntos de datos y realizar operaciones complejas con una mínima cantidad de código. Se puede responder casi cualquier pregunta y realizar cualquier análisis, siempre y cuando se hayan recopilado los datos de respaldo y comprenda cómo construir la consulta adecuada. Para más información, consulte [Introducción a las consultas de registro en Azure Monitor](../azure-monitor/logs/get-started-queries.md).

1. En el **área de trabajo de Log Analytics**, seleccione **Registros**.
1. En el editor de consultas, pegue la siguiente consulta del [lenguaje de consulta Kusto](/azure/data-explorer/kusto/query/). Esta consulta muestra el uso de la directiva por operación durante los últimos X días. El período predeterminado está establecido en 90 días (90d). Tenga en cuenta que la consulta solo se centra en operaciones en las que se emite un token o código mediante la directiva.

    ```kusto
    AuditLogs
    | where TimeGenerated  > ago(90d)
    | where OperationName contains "issue"
    | extend  UserId=extractjson("$.[0].id",tostring(TargetResources))
    | extend Policy=extractjson("$.[1].value",tostring(AdditionalDetails))
    | summarize SignInCount = count() by Policy, OperationName
    | order by SignInCount desc  nulls last
    ```

1. Seleccione **Run** (Ejecutar). Los resultados de la consulta se muestran en la parte inferior de la pantalla.
1. Para guardar la consulta para su uso posterior, seleccione **Guardar**.

   ![Editor de registros de Log Analytics](./media/azure-monitor/query-policy-usage.png)

1. Rellene la siguiente información:

    - **Nombre**: escriba el nombre de la consulta.
    - **Guardar como**: seleccione `query`.
    - **Categoría**: seleccione `Log`.

1. Seleccione **Guardar**.

También puede cambiar la consulta para visualizar los datos mediante el operador [render](/azure/data-explorer/kusto/query/renderoperator?pivots=azuremonitor).

```kusto
AuditLogs
| where TimeGenerated  > ago(90d)
| where OperationName contains "issue"
| extend  UserId=extractjson("$.[0].id",tostring(TargetResources))
| extend Policy=extractjson("$.[1].value",tostring(AdditionalDetails))
| summarize SignInCount = count() by Policy
| order by SignInCount desc  nulls last
| render  piechart
```

![Gráfico circular del editor de registros de Log Analytics](./media/azure-monitor/query-policy-usage-pie.png)

Para obtener más ejemplos, consulte el [repositorio de GitHub SIEM](https://aka.ms/b2csiem) de Azure AD B2C.

### <a name="62-create-a-workbook"></a>6.2 Creación de un libro

Los libros proporcionan un lienzo flexible para el análisis de datos y la creación de informes visuales completos en el Azure Portal. Permiten acceder a varios orígenes de datos desde Azure y combinarlos en experiencias interactivas unificadas. Para más información, consulte [Libros de Azure Monitor](../azure-monitor/visualize/workbooks-overview.md).

Siga las instrucciones que se indican a continuación para crear un nuevo libro mediante una plantilla de la galería de JSON. En este libro se proporciona un panel de **Información del usuario** y **Autenticación** para el inquilino de Azure AD B2C.

1. En el **área de trabajo de Log Analytics**, seleccione **Libros**.
1. En la barra de herramientas, seleccione la opción **+ Nuevo** para crear un nuevo libro.
1. En la página **Nuevo libro**, seleccione el **Editor avanzado** mediante la opción **</>** de la barra de herramientas.

     ![Plantilla de la galería](./media/azure-monitor/wrkb-adv-editor.png)

1. Seleccione **Plantilla de la galería**.
1. Reemplace el archivo JSON de la **plantilla de la galería**  por el contenido del [libro básico de Azure AD B2C](https://raw.githubusercontent.com/azure-ad-b2c/siem/master/workbooks/dashboard.json):
1. Aplique la plantilla mediante el botón **Aplicar**.
1. Seleccione **Edición finalizada** en la barra de herramientas para finalizar la edición del libro.
1. Por último, guarde el libro mediante el botón **Guardar** de la barra de herramientas.
1. Proporcione un **título**, como *Panel de Azure AD B2C*.
1. Seleccione **Guardar**.

    ![Guardar el libro](./media/azure-monitor/wrkb-title.png)

El libro mostrará los informes en forma de panel.

![Primer panel del libro](./media/azure-monitor/wkrb-dashboard-1.png)

![Segundo panel del libro](./media/azure-monitor/wrkb-dashboard-2.png)

![Tercer panel del libro](./media/azure-monitor/wrkb-dashboard-3.png)


## <a name="create-alerts"></a>Creación de alertas

Las alertas se crean mediante reglas de alertas en Azure Monitor y pueden ejecutar automáticamente consultas guardadas o búsquedas de registros personalizadas a intervalos regulares. Puede crear alertas basadas en métricas de rendimiento específicas o cuando se creen determinados eventos, haya un evento ausente o se cree una serie de eventos dentro de una ventana de tiempo determinada. Por ejemplo, las alertas se pueden usar para recibir una notificación cuando el número medio de inicios de sesión supere un determinado umbral. Para más información, consulte [Creación de alertas](../azure-monitor/alerts/alerts-log.md).


Use las siguientes instrucciones para crear una nueva alerta de Azure, que enviará una [notificación de correo electrónico](../azure-monitor/alerts/action-groups.md#configure-notifications) siempre que haya un descenso del 25 % en el **total de solicitudes** en comparación con el período anterior. La alerta se ejecutará cada 5 minutos y buscará el descenso en períodos que comprenden las últimas 24 horas. Las alertas se crean mediante el lenguaje de consulta Kusto.


1. En el **área de trabajo de Log Analytics**, seleccione **Registros**. 
1. Use la siguiente consulta para crear una nueva **consulta de Kusto**.

    ```kusto
    let start = ago(24h);
    let end = now();
    let threshold = -25; //25% decrease in total requests.
    AuditLogs
    | serialize TimeGenerated, CorrelationId, Result
    | make-series TotalRequests=dcount(CorrelationId) on TimeGenerated in range(start, end, 1h)
    | mvexpand TimeGenerated, TotalRequests
    | where TotalRequests > 0
    | serialize TotalRequests, TimeGenerated, TimeGeneratedFormatted=format_datetime(todatetime(TimeGenerated), 'yyyy-M-dd [hh:mm:ss tt]')
    | project   TimeGeneratedFormatted, TotalRequests, PercentageChange= ((toreal(TotalRequests) - toreal(prev(TotalRequests,1)))/toreal(prev(TotalRequests,1)))*100
    | order by TimeGeneratedFormatted
    | where PercentageChange <= threshold   //Trigger's alert rule if matched.
    ```

1. Seleccione **Ejecutar** para probar la consulta. Debería ver los resultados si hay un descenso del 25 % o más en el total de solicitudes en las últimas 24 horas.
1. Para crear una regla de alerta basada en la consulta anterior, use la opción **+ Nueva regla de alertas** disponible en la barra de herramientas.
1. En la página **Crear regla de alerta**, seleccione **Nombre de la condición**. 
1. En la página **Configurar la lógica de la señal**, establezca los valores siguientes y, a continuación, use el botón **Listo** para guardar los cambios.
    * Lógica de alerta: Establezca **Número de resultados** **mayor que** **0**.
    * Evaluación basada en: Seleccione **1440** para el período (en minutos) y **5** para la frecuencia (en minutos). 

    ![Creación de una condición de regla de alerta](./media/azure-monitor/alert-create-rule-condition.png)

Una vez creada la alerta, vaya al **área de trabajo de Log Analytics** y seleccione **Alertas**. En esta página se muestran todas las alertas que se han desencadenado en el período establecido por la opción **Intervalo de tiempo**.  

### <a name="configure-action-groups"></a>Configuración de grupos de acciones

Las alertas de Azure Monitor y Service Health usan grupos de acciones para notificar a los usuarios que se ha desencadenado una alerta. Puede incluir el envío de llamadas de voz, mensajes de texto o correo electrónico o el desencadenamiento de varios tipos de acciones automatizadas. Siga la guía [Creación y administración de grupos de acciones en Azure Portal](../azure-monitor/alerts/action-groups.md).

A continuación se incluye un ejemplo de un correo electrónico de notificación de alerta. 

   ![Notificación por correo electrónico](./media/azure-monitor/alert-email-notification.png)

## <a name="multiple-tenants"></a>Varios inquilinos

Para incorporar varios registros de inquilino de Azure AD B2C a la misma área de trabajo de Log Analytics (o a una cuenta de Azure Storage o instancia de Event Hubs), necesitará implementaciones independientes con distintos valores de **nombre de oferta de MSP**. Asegúrese de que el área de trabajo de Log Analytics se encuentra en el mismo grupo de recursos que el que ha configurado en [Crear o elegir un grupo de recursos](#1-create-or-choose-resource-group).

Cuando trabaje con varias áreas de trabajo de Log Analytics, use [consultas entre áreas de trabajo](../azure-monitor/logs/cross-workspace-query.md) para crear consultas que funcionen en varias áreas de trabajo. Por ejemplo, la consulta siguiente realiza una unión de dos registros de auditoría de distintos inquilinos basados en la misma categoría (por ejemplo, autenticación):

```kusto
workspace("AD-B2C-TENANT1").AuditLogs
| join  workspace("AD-B2C-TENANT2").AuditLogs
  on $left.Category== $right.Category
```

## <a name="change-the-data-retention-period"></a>Cambio del período de retención de datos

Los registros de Azure Monitor están diseñados para escalar y admitir la recopilación, indexación y almacenamiento de grandes cantidades de datos por día provenientes de cualquier origen dentro de su empresa o implementados en Azure. De forma predeterminada, los registros se conservan durante 30 días, pero la duración de la retención se puede aumentar hasta 2 años. Obtenga información acerca de cómo [administrar el uso y los costos con los registros de Azure Monitor](../azure-monitor/logs/manage-cost-storage.md). Después de seleccionar el plan de tarifa, puede [cambiar el período de retención de datos](../azure-monitor/logs/manage-cost-storage.md#change-the-data-retention-period).

## <a name="next-steps"></a>Pasos siguientes

* Consulte más ejemplos en la [galería de SIEM](https://aka.ms/b2csiem) de Azure AD B2C. 

* Para obtener más información sobre cómo agregar y establecer la configuración de diagnóstico en Azure Monitor, consulte [Tutorial: Recopilación y análisis de registros de recurso desde un recurso de Azure](../azure-monitor/essentials/monitor-azure-resource.md).

* Para obtener información sobre la transmisión de registros de Azure AD a un centro de eventos, consulte [Tutorial: Transmisión de registros de Azure Active Directory a un centro de eventos de Azure](../active-directory/reports-monitoring/tutorial-azure-monitor-stream-logs-to-event-hub.md).
