---
title: Supervisión del estado de inicio de sesión de la aplicación para la resistencia en Azure Active Directory
description: Cree consultas y notificaciones para supervisar el estado de inicio de sesión de las aplicaciones.
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 03/17/2021
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 47691ae404f65f04ace36485cb01fc5617d00a9a
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105031773"
---
# <a name="monitoring-application-sign-in-health-for-resilience"></a>Supervisión del estado de inicio de sesión de la aplicación para la resistencia

Para aumentar la resistencia de la infraestructura, configure la supervisión del estado de inicio de sesión de la aplicación para las aplicaciones críticas, de modo que se reciba una alerta si se produce un incidente con impacto. Para ayudarle en esta iniciativa, puede configurar las alertas basadas en el libro de estados de inicio de sesión. 

Este libro permite a los administradores supervisar las solicitudes de autenticación de las aplicaciones en el inquilino. Ofrece estas funcionalidades principales:

* Configure el libro para supervisar las aplicaciones juntas o de manera individual con datos casi en tiempo real.

* Configure alertas para recibir una notificación cuando cambien los patrones de autenticación, a fin de que pueda investigar y tomar medidas.

* Compare las tendencias en un período; por ejemplo, semana a semana, que es la configuración predeterminada del libro.

> [!NOTE]
> Para ver todos los libros disponibles y los requisitos previos para usarlos, consulte [Uso de los libros de Azure Monitor en informes](../reports-monitoring/howto-use-azure-monitor-workbooks.md).

Durante un evento con impacto, pueden suceder dos cosas:

* El número de inicios de sesión de una aplicación puede reducirse de manera precipitada porque los usuarios no pueden iniciar sesión.

* El número de errores de inicio de sesión puede aumentar. 

En este artículo se explica cómo configurar el libro de estado de inicio de sesión para supervisar las interrupciones de los inicios de sesión de los usuarios.

## <a name="prerequisites"></a>Prerrequisitos

* Un inquilino de Azure AD.

* Un usuario con el rol de administrador global o administrador de seguridad para el inquilino de Azure AD.

* Un área de trabajo de Log Analytics en la suscripción a Azure para enviar registros a los registros de Azure Monitor. 

   * Aprenda a [crear un área de trabajo de Log Analytics](../../azure-monitor/logs/quick-create-workspace.md)

* Registros de Azure AD integrados con los registros de Azure Monitor.

   * Aprenda a [integrar registros de inicio de sesión de Azure AD con la transmisión de Azure Monitor.](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

## <a name="configure-the-app-sign-in-health-workbook"></a>Configuración del libro de estado de inicio de sesión de la aplicación 

Para acceder a los libros, abra **Azure Portal**, seleccione **Azure Active Directory** y, a continuación, seleccione **Libros**.

Verá los libros en Uso, Acceso condicional y Solucionar problemas. El libro de estado de inicio de sesión de la aplicación se muestra en la sección Uso.

Una vez que se usa un libro, puede aparecer en la sección Libros modificados recientemente.

![Captura de pantalla que muestra la galería de libros en Azure Portal.](./media/monitor-sign-in-health-for-resilience/sign-in-health-workbook.png)


El libro de estado de inicio de sesión de la aplicación permite visualizar lo que sucede con los inicios de sesión. 

De manera predeterminada, el libro presenta dos grafos. Estos grafos comparan lo que sucede en las aplicaciones ahora con el mismo período hace una semana. Las líneas azules son la semana actual y las líneas naranja son la semana anterior.

![Captura de pantalla que muestra los grafos de estado de inicio de sesión.](./media/monitor-sign-in-health-for-resilience/sign-in-health-graphs.png)

**El primer grafo es el Uso por hora (número de usuarios correctos)** . Comparar el número actual de usuarios correctos en un período de uso típico le ayuda a detectar una reducción en el uso que puede requerir investigación. Una reducción de la frecuencia de uso correcto puede ayudar a detectar los problemas de rendimiento y uso que no se pueden detectar con la frecuencia de errores. Por ejemplo, si los usuarios no pueden comunicarse con la aplicación para intentar iniciar sesión, no habrá ningún error, solo una reducción del uso. En la sección siguiente se puede ver una consulta de ejemplo para estos datos.

**El segundo grafo es la Frecuencia de errores por hora**. Un pico en la frecuencia de errores puede indicar un problema con los mecanismos de autenticación. La frecuencia de errores solo puede medirse si los usuarios pueden tratar de realizar la autenticación. Si los usuarios no pueden acceder para realizar el intento, no se mostrarán los errores.

Puede configurar una alerta que notifique a un grupo específico cuando la frecuencia de uso o errores supere un umbral especificado. En la sección siguiente se puede ver una consulta de ejemplo para estos datos.

## <a name="configure-the-query-and-alerts"></a>Configuración de la consulta y las alertas

Cree reglas de alertas en Azure Monitor y puede ejecutar automáticamente consultas guardadas o búsquedas de registros personalizadas a intervalos regulares.

Use las siguientes instrucciones para crear alertas por correo electrónico basadas en las consultas reflejadas en los grafos. Los siguientes scripts de ejemplo enviarán una notificación por correo electrónico cuando

* el uso correcto se reduzca en un 90 % en comparación con la misma hora hace dos días, como en el grafo de uso por hora de la sección anterior. 

* la frecuencia de errores aumente en un 90 % en comparación con la misma hora hace dos días, como en el grafo de frecuencia de errores por hora de la sección anterior. 

 Para configurar la consulta subyacente y establecer alertas, realice los pasos siguientes. Usará la consulta de ejemplo como base para realizar su configuración. Al final de esta sección se encuentra una explicación de la estructura de la consulta.

Para obtener más información sobre cómo crear, ver y administrar alertas de registro mediante Azure Monitor, consulte [Administración de alertas de registro](../../azure-monitor/alerts/alerts-log.md).

1. En el libro, seleccione **Editar** y, a continuación, seleccione el **icono de consulta** sobre el lado derecho del grafo.   

   [![Captura de pantalla que muestra cómo editar el libro.](./media/monitor-sign-in-health-for-resilience/edit-workbook.png)](./media/monitor-sign-in-health-for-resilience/edit-workbook.png)

   Se abre el registro de consultas.

   [![Captura de pantalla en la que se muestra el registro de consultas.](./media/monitor-sign-in-health-for-resilience/query-log.png)](./media/monitor-sign-in-health-for-resilience/query-log.png)
‎

2. Copie uno de los scripts de ejemplo para una nueva consulta de Kusto.  
   * [Consulta de Kusto para el aumento de la frecuencia de errores](#kusto-query-for-increase-in-failure-rate)
   * [Consulta de Kusto para la reducción de uso](#kusto-query-for-drop-in-usage)

3. Pegue la consulta en la ventana y seleccione **Ejecutar**. Asegúrese de que puede ver el mensaje de completado que se muestra en la imagen siguiente y los resultados por debajo de dicho mensaje.

   [![Captura de pantalla que muestra los resultados de la ejecución de la consulta.](./media/monitor-sign-in-health-for-resilience/run-query.png)](./media/monitor-sign-in-health-for-resilience/run-query.png)

4. Resalte la consulta y seleccione + **Nueva regla de alerta**. 
 
   [![Captura de pantalla que muestra la pantalla de nueva regla de alerta.](./media/monitor-sign-in-health-for-resilience/new-alert-rule.png)](./media/monitor-sign-in-health-for-resilience/new-alert-rule.png)


5. Configure las condiciones de la alerta. En la sección Condición, seleccione el vínculo **cuando el promedio de búsqueda de registros personalizados sea mayor que el número definido por la lógica**. En el panel Configurar lógica de señal, desplácese a Lógica de alerta.

   [![Captura de pantalla que muestra la pantalla de configuración de alertas.](./media/monitor-sign-in-health-for-resilience/configure-alerts.png)](./media/monitor-sign-in-health-for-resilience/configure-alerts.png)
 
   * **Valor del umbral**: 0. Este valor generará una alerta sobre cualquier resultado.

   * **Período de evaluación (en minutos)** : 2880. Este valor revisa una hora.

   * **Frecuencia (en minutos)** : 60. Este valor establece el período de evaluación en una vez cada hora durante la hora anterior.

   * Seleccione **Listo**.

6. En la sección **Acciones**, configure estas opciones:  

    [![Captura de pantalla que muestra la página Crear regla de alertas.](./media/monitor-sign-in-health-for-resilience/create-alert-rule.png)](./media/monitor-sign-in-health-for-resilience/create-alert-rule.png)

   * En **Acciones**, elija **Seleccionar grupo de acciones** y agregue el grupo sobre el que quiere recibir alertas.

   * En **Personalizar acciones**, seleccione **Alertas de correo electrónico**.

   * Agregue una **línea de asunto**.

7. En **Detalles de la regla de alertas**, configure estas opciones:

   * Agregue un nombre descriptivo y una descripción.

   * Seleccione el **grupo de recursos** al que quiere agregar la alerta.

   * Seleccione la **gravedad** predeterminada de la alerta.

   * Seleccione **Habilitar regla de alertas tras la creación** si quiere activarla inmediatamente; de lo contrario, seleccione **Suprimir alertas**.

8. Seleccione **Crear regla de alertas**.

9. Seleccione **Guardar**, escriba un nombre para la consulta y seleccione **Guardar como consulta con una categoría de alerta**. Después, seleccione **Guardar** de nuevo.  

   [![Captura de pantalla que muestra el botón Guardar consulta.](./media/monitor-sign-in-health-for-resilience/save-query.png)](./media/monitor-sign-in-health-for-resilience/save-query.png)

### <a name="refine-your-queries-and-alerts"></a>Restricción de las consultas y alertas

Modifique las consultas y alertas para obtener la máxima eficacia.

* Asegúrese de probar las alertas.

* Modifique la sensibilidad y la frecuencia de las alertas para recibir notificaciones importantes. Los administradores se pueden insensibilizar a las alertas si reciben demasiadas y omiten algo importante. 

* Asegúrese de que el correo electrónico del que proceden las alertas esté incluido en la lista de remitentes permitidos de los clientes de correo electrónico del administrador. De lo contrario, es posible que se omitan notificaciones debido a un filtro de correo no deseado en el cliente de correo electrónico. 

* La consulta de alertas en Azure Monitor solo puede incluir resultados de las últimas 48 horas. [Esta es una limitación actual por diseño](https://github.com/MicrosoftDocs/azure-docs/issues/22637).

## <a name="sample-scripts"></a>Muestras de scripts

### <a name="kusto-query-for-increase-in-failure-rate"></a>Consulta de Kusto para el aumento de la frecuencia de errores

   La frecuencia de la parte inferior se puede ajustar según sea necesario y representa el cambio porcentual del tráfico en la última hora en comparación con la misma hora ayer. 0,5 significa que hay una diferencia del 50 % en el tráfico.

```kusto

let today = SigninLogs

| where TimeGenerated > ago(1h) // Query failure rate in the last hour
 
| project TimeGenerated, UserPrincipalName, AppDisplayName, status = case(Status.errorCode == "0", "success", "failure")

// Optionally filter by a specific application

//| where AppDisplayName == **APP NAME**

| summarize success = countif(status == "success"), failure = countif(status == "failure") by bin(TimeGenerated, 1h) // hourly failure rate

| project TimeGenerated, failureRate = (failure * 1.0) / ((failure + success) * 1.0)

| sort by TimeGenerated desc

| serialize rowNumber = row_number();

let yesterday = SigninLogs

| where TimeGenerated between((ago(1h) - totimespan(1d))..(now() - totimespan(1d))) // Query failure rate at the same time yesterday

| project TimeGenerated, UserPrincipalName, AppDisplayName, status = case(Status.errorCode == "0", "success", "failure")

// Optionally filter by a specific application

//| where AppDisplayName == **APP NAME**

| summarize success = countif(status == "success"), failure = countif(status == "failure") by bin(TimeGenerated, 1h) // hourly failure rate at same time yesterday

| project TimeGenerated, failureRateYesterday = (failure * 1.0) / ((failure + success) * 1.0)

| sort by TimeGenerated desc

| serialize rowNumber = row_number();
today
| join (yesterday) on rowNumber // join data from same time today and yesterday

| project TimeGenerated, failureRate, failureRateYesterday

// Set threshold to be the percent difference in failure rate in the last hour as compared to the same time yesterday

| where abs(failureRate - failureRateYesterday) > 0.5

```

### <a name="kusto-query-for-drop-in-usage"></a>Consulta de Kusto para la reducción de uso

En la siguiente consulta se compara el tráfico en la última hora con el de la misma hora ayer.
Se excluyen los días sábado, domingo y lunes, ya que se espera que en esos días haya una gran variabilidad de tráfico respecto a la misma hora el día anterior. 

La frecuencia de la parte inferior se puede ajustar según sea necesario y representa el cambio porcentual del tráfico en la última hora en comparación con la misma hora ayer. 0,5 significa que hay una diferencia del 50 % en el tráfico.

*Debe ajustar estos valores para que se adapten al modelo de operación de su empresa*.

```Kusto
 let today = SigninLogs // Query traffic in the last hour

| where TimeGenerated > ago(1h)

| project TimeGenerated, AppDisplayName, UserPrincipalName

// Optionally filter by AppDisplayName to scope query to a single application

//| where AppDisplayName contains "Office 365 Exchange Online"

| summarize users = dcount(UserPrincipalName) by bin(TimeGenerated, 1hr) // Count distinct users in the last hour

| sort by TimeGenerated desc

| serialize rn = row_number();

let yesterday = SigninLogs // Query traffic at the same hour yesterday

| where TimeGenerated between((ago(1h) - totimespan(1d))..(now() - totimespan(1d))) // Count distinct users in the same hour yesterday

| project TimeGenerated, AppDisplayName, UserPrincipalName

// Optionally filter by AppDisplayName to scope query to a single application

//| where AppDisplayName contains "Office 365 Exchange Online"

| summarize usersYesterday = dcount(UserPrincipalName) by bin(TimeGenerated, 1hr)

| sort by TimeGenerated desc

| serialize rn = row_number();

today
| join // Join data from today and yesterday together
(
yesterday
)
on rn

// Calculate the difference in number of users in the last hour compared to the same time yesterday

| project TimeGenerated, users, usersYesterday, difference = abs(users - usersYesterday), max = max_of(users, usersYesterday)

 extend ratio = (difference * 1.0) / max // Ratio is the percent difference in traffic in the last hour as compared to the same time yesterday

// Day variable is the number of days since the previous Sunday. Optionally ignore results on Sat, Sun, and Mon because large variability in traffic is expected.

| extend day = dayofweek(now())

| where day != time(6.00:00:00) // exclude Sat

| where day != time(0.00:00:00) // exclude Sun

| where day != time(1.00:00:00) // exclude Mon

| where ratio > 0.7 // Threshold percent difference in sign-in traffic as compared to same hour yesterday

```

## <a name="create-processes-to-manage-alerts"></a>Creación de procesos para administrar alertas

Una vez que haya configurado la consulta y las alertas, cree los procesos empresariales para administrar las alertas.

* ¿Quién supervisará el libro y cuándo?

* Cuando se genere una alerta, ¿quién la investigará?

* ¿Cuáles son las necesidades de comunicación? ¿Quién creará las comunicaciones y quién las recibirá?

* Si se produce una interrupción, ¿qué procesos empresariales deben desencadenarse?

## <a name="next-steps"></a>Pasos siguientes

[Más información acerca de los libros](../reports-monitoring/howto-use-azure-monitor-workbooks.md)
