---
title: Desencadenamiento de acciones complejas con alertas de Azure Monitor
description: Obtenga información sobre cómo crear una acción de aplicación lógica para procesar las alertas de Azure Monitor.
author: dkamstra
ms.author: dukek
ms.topic: conceptual
ms.date: 02/19/2021
ms.openlocfilehash: f1e81dca6926ae9f57e428eb1cef761c588a78b6
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107029852"
---
# <a name="how-to-trigger-complex-actions-with-azure-monitor-alerts"></a>Procedimientos para desencadenar acciones complejas con alertas de Azure Monitor

En este artículo se muestra cómo configurar y desencadenar una aplicación lógica para crear una conversación en Microsoft Teams cuando se activa una alerta.

## <a name="overview"></a>Información general

Cuando se desencadena una alerta de Azure Monitor, llama a un [grupo de acciones](./action-groups.md). Los grupos de acciones permiten desencadenar una o varias acciones para notificar sobre la alerta a otros usuarios y también a resolverla.

El proceso general es:

-   Crear la aplicación lógica para el tipo de alerta respectivo.

-   Importar una carga de ejemplo para el tipo de alerta respectivo en la aplicación lógica.

-   Definir el comportamiento de la aplicación lógica.

-   Copiar el punto de conexión HTTP de la aplicación lógica en un grupo de acciones de Azure.

El proceso es similar si quiere que la aplicación lógica lleve a cabo otra acción.

## <a name="create-an-activity-log-alert-administrative"></a>Creación de una alerta del Registro de actividad: Administrativo

1. [Crear una aplicación lógica](~/articles/logic-apps/quickstart-create-first-logic-app-workflow.md).

1.  Seleccione el desencadenador: **Cuando se recibe una solicitud HTTP**.

1. En el cuadro de diálogo **Cuando se recibe una solicitud HTTP**, seleccione **Usar una carga de ejemplo para generar el esquema**.

    ![Instantánea en la que se muestra el cuadro de diálogo "Cuando se recibe una solicitud H T T P" con la opción "Usar una carga de ejemplo para generar el esquema" seleccionada. ](~/articles/app-service/media/tutorial-send-email/generate-schema-with-payload.png)

1.  Copie y pegue la carga de ejemplo siguiente en el cuadro de diálogo:

    ```json
        {
            "schemaId&quot;: &quot;Microsoft.Insights/activityLogs&quot;,
            &quot;data&quot;: {
                &quot;status&quot;: &quot;Activated&quot;,
                &quot;context&quot;: {
                &quot;activityLog&quot;: {
                    &quot;authorization&quot;: {
                    &quot;action&quot;: &quot;microsoft.insights/activityLogAlerts/write&quot;,
                    &quot;scope&quot;: &quot;/subscriptions/…&quot;
                    },
                    &quot;channels&quot;: &quot;Operation&quot;,
                    &quot;claims&quot;: &quot;…&quot;,
                    &quot;caller&quot;: &quot;logicappdemo@contoso.com&quot;,
                    &quot;correlationId&quot;: &quot;91ad2bac-1afa-4932-a2ce-2f8efd6765a3&quot;,
                    &quot;description&quot;: &quot;&quot;,
                    &quot;eventSource&quot;: &quot;Administrative&quot;,
                    &quot;eventTimestamp&quot;: &quot;2018-04-03T22:33:11.762469+00:00&quot;,
                    &quot;eventDataId&quot;: &quot;ec74c4a2-d7ae-48c3-a4d0-2684a1611ca0&quot;,
                    &quot;level&quot;: &quot;Informational&quot;,
                    &quot;operationName&quot;: &quot;microsoft.insights/activityLogAlerts/write&quot;,
                    &quot;operationId&quot;: &quot;61f59fc8-1442-4c74-9f5f-937392a9723c&quot;,
                    &quot;resourceId&quot;: &quot;/subscriptions/…&quot;,
                    &quot;resourceGroupName&quot;: &quot;LOGICAPP-DEMO&quot;,
                    &quot;resourceProviderName&quot;: &quot;microsoft.insights&quot;,
                    &quot;status&quot;: &quot;Succeeded&quot;,
                    &quot;subStatus&quot;: &quot;&quot;,
                    &quot;subscriptionId&quot;: &quot;…&quot;,
                    &quot;submissionTimestamp&quot;: &quot;2018-04-03T22:33:36.1068742+00:00&quot;,
                    &quot;resourceType&quot;: &quot;microsoft.insights/activityLogAlerts&quot;
                }
                },
                &quot;properties&quot;: {}
            }
        }
    ```

1. Se abrirá una ventana emergente en el **diseñador de aplicaciones lógicas** para recordarle que la solicitud enviada a la aplicación lógica debe establecer el encabezado **Content-Type** en **application/json**. Cierre la ventana emergente. La alerta de Azure Monitor establece el encabezado.

    ![Establecimiento del encabezado Content-Type](media/action-groups-logic-app/content-type-header.png &quot;Establecimiento del encabezado Content-Type")

1. Seleccione **+** **Nuevo paso** y luego elija **Agregar una acción**.

    ![Agregar una acción](media/action-groups-logic-app/add-action.png "Agregar una acción")

1. Busque y seleccione el conector de Microsoft Teams. Elija la acción **Microsoft Teams – Post message** (Microsoft Teams: publicar mensaje).

    ![Acciones de Microsoft Teams](media/action-groups-logic-app/microsoft-teams-actions.png "Acciones de Microsoft Teams")

1. Configure la acción de Microsoft Teams. El **diseñador de aplicaciones lógicas** le pide que se autentique en su cuenta profesional o educativa. Elija el **Id. de equipo** y el **Id. de canal** para enviar el mensaje.

13. Configure el mensaje con una combinación de texto estático y referencias a los \<fields\> del contexto dinámico. Corte y pegue el texto siguiente en el campo **Mensaje**:

    ```text
      Activity Log Alert: <eventSource>
      operationName: <operationName>
      status: <status>
      resourceId: <resourceId>
    ```

    Luego busque y reemplace los \<fields\> por etiquetas de contenido dinámico con el mismo nombre.

    > [!NOTE]
    > Hay dos campos dinámicos denominados **status**. Agregue ambos campos al mensaje. Use el campo del contenedor de propiedades **activityLog** y elimine el otro. Si pasa el puntero sobre el campo **status**, verá la referencia de campo completa como se muestra en la captura de pantalla siguiente:

    ![Acción de Microsoft Teams: Publicar un mensaje](media/action-groups-logic-app/teams-action-post-message.png "Acción de Microsoft Teams: publicar un mensaje")

1. En la parte superior del **diseñador de Logic Apps**, seleccione **Guardar** para guardar la aplicación lógica.

1. Abra el grupo de acciones existente y agregue una acción para hacer referencia a la aplicación lógica. Si no existe un grupo de acciones, consulte [Creación y administración de grupos de acciones en Azure Portal](./action-groups.md) para crear uno. No olvide guardar los cambios.

    ![Actualización del grupo de acciones](media/action-groups-logic-app/update-action-group.png "Actualización del grupo de acciones")

La próxima vez que una alerta llame al grupo de acciones, se llamará a la aplicación lógica.

## <a name="create-a-service-health-alert"></a>Creación de una alerta de Service Health

Las entradas de Azure Service Health forman parte del registro de actividad. El proceso de creación de la alerta es similar a la [creación de una alerta de registro de actividad](#create-an-activity-log-alert-administrative), pero con unos cuantos cambios:

- Los pasos del 1 al 3 son iguales.
- Para el paso 4, use la carga de ejemplo siguiente para el desencadenador de solicitud HTTP:

    ```json
    {
        "schemaId": "Microsoft.Insights/activityLogs",
        "data": {
            "status": "Activated",
            "context": {
                "activityLog": {
                    "channels": "Admin",
                    "correlationId": "e416ed3c-8874-4ec8-bc6b-54e3c92a24d4",
                    "description": "…",
                    "eventSource": "ServiceHealth",
                    "eventTimestamp": "2018-04-03T22:44:43.7467716+00:00",
                    "eventDataId": "9ce152f5-d435-ee31-2dce-104228486a6d",
                    "level": "Informational",
                    "operationName": "Microsoft.ServiceHealth/incident/action",
                    "operationId": "e416ed3c-8874-4ec8-bc6b-54e3c92a24d4",
                    "properties": {
                        "title": "...",
                        "service": "...",
                        "region": "Global",
                        "communication": "...",
                        "incidentType": "Incident",
                        "trackingId": "...",
                        "impactStartTime": "2018-03-22T21:40:00.0000000Z",
                        "impactMitigationTime": "2018-03-22T21:41:00.0000000Z",
                        "impactedServices": "[{\"ImpactedRegions\"}]",
                        "defaultLanguageTitle": "...",
                        "defaultLanguageContent": "...",
                        "stage": "Active",
                        "communicationId": "11000001466525",
                        "version": "0.1.1"
                    },
                    "status": "Active",
                    "subscriptionId": "...",
                    "submissionTimestamp": "2018-04-03T22:44:50.8013523+00:00"
                }
            },
            "properties": {}
        }
    }
    ```

-  Los pasos 5 y 6 son iguales.
-  Para los pasos 7 a 11, siga este procedimiento:

   1. Seleccione **+** **Nuevo paso** y luego elija **Agregar una condición**. Establezca las condiciones siguientes para que la aplicación lógica solo se ejecute cuando los datos de entrada coincidan con los siguientes.  Cuando especifique el valor de la versión en el cuadro de texto, inclúyalo entre comillas ("0.1.1") para asegurarse de que se evalúa como cadena y no como tipo numérico.  El sistema no muestra las comillas si vuelve a la página, pero el código subyacente conserva el tipo de cadena.   
       - `schemaId == Microsoft.Insights/activityLogs`
       - `eventSource == ServiceHealth`
       - `version == "0.1.1"`

      !["Condición de carga de Service Health"](media/action-groups-logic-app/service-health-payload-condition.png "Condición de carga de Service Health")

   1. En la condición **if true**, siga las instrucciones que se indican en los pasos 11 a 13 de [Creación de una alerta del registro de actividad](#create-an-activity-log-alert-administrative) para agregar la acción de Microsoft Teams.

   1. Defina el mensaje con una combinación de HTML y contenido dinámico. Corte y pegue el texto siguiente en el campo **Mensaje**. Reemplace los campos `[incidentType]`, `[trackingID]`, `[title]` y `[communication]` por etiquetas de contenido dinámico con el mismo nombre:

       ```html
       <p>
       <b>Alert Type:</b>&nbsp;<strong>[incidentType]</strong>&nbsp;
       <strong>Tracking ID:</strong>&nbsp;[trackingId]&nbsp;
       <strong>Title:</strong>&nbsp;[title]</p>
       <p>
       <a href="https://ms.portal.azure.com/#blade/Microsoft_Azure_Health/AzureHealthBrowseBlade/serviceIssues">For details, log in to the Azure Service Health dashboard.</a>
       </p>
       <p>[communication]</p>
       ```

       !["Acción posterior a la condición true de Service Health"](media/action-groups-logic-app/service-health-true-condition-post-action.png "Acción posterior a la condición true de Service Health")

   1. Para la condición **If false**, proporcione un mensaje útil:

       ```html
       <p><strong>Service Health Alert</strong></p>
       <p><b>Unrecognized alert schema</b></p>
       <p><a href="https://ms.portal.azure.com/#blade/Microsoft_Azure_Health/AzureHealthBrowseBlade/serviceIssues">For details, log in to the Azure Service Health dashboard.\</a></p>
       ```

       !["Acción posterior a la condición false de Service Health"](media/action-groups-logic-app/service-health-false-condition-post-action.png "Acción posterior a la condición false de Service Health")

- El paso 15 es igual. Siga las instrucciones para guardar la aplicación lógica y actualizar el grupo de acciones.

## <a name="create-a-metric-alert"></a>Creación de una alerta de métrica

El proceso de creación de una alerta de métrica es similar a la [creación de una alerta de registro de actividad](#create-an-activity-log-alert-administrative), pero con unos cuantos cambios:

- Los pasos del 1 al 3 son iguales.
- Para el paso 4, use la carga de ejemplo siguiente para el desencadenador de solicitud HTTP:

    ```json
    {
    "schemaId": "AzureMonitorMetricAlert",
    "data": {
        "version": "2.0",
        "status": "Activated",
        "context": {
        "timestamp": "2018-04-09T19:00:07.7461615Z",
        "id": "...",
        "name": "TEST-VM CPU Utilization",
        "description": "",
        "conditionType": "SingleResourceMultipleMetricCriteria",
        "condition": {
            "windowSize": "PT15M",
            "allOf": [
                {
                    "metricName": "Percentage CPU",
                    "dimensions": [
                    {
                        "name": "ResourceId",
                        "value": "d92fc5cb-06cf-4309-8c9a-538eea6a17a6"
                    }
                ],
                "operator": "GreaterThan",
                "threshold": "5",
                "timeAggregation": "PT15M",
                "metricValue": 1.0
            }
            ]
        },
        "subscriptionId": "...",
        "resourceGroupName": "TEST",
        "resourceName": "test-vm",
        "resourceType": "Microsoft.Compute/virtualMachines",
        "resourceId": "...",
        "portalLink": "..."
        },
        "properties": {}
    }
    }
    ```

- Los pasos 5 y 6 son iguales.
- Para los pasos 7 a 11, siga este procedimiento:

  1. Seleccione **+** **Nuevo paso** y luego elija **Agregar una condición**. Establezca las condiciones siguientes para que la aplicación lógica solo se ejecute cuando los datos de entrada coincidan con los siguientes. Cuando especifique el valor de la versión en el cuadro de texto, inclúyalo entre comillas ("2.0") para asegurarse de que se evalúa como cadena y no como tipo numérico.  El sistema no muestra las comillas si vuelve a la página, pero el código subyacente conserva el tipo de cadena. 
     - `schemaId == AzureMonitorMetricAlert`
     - `version == "2.0"`
       
       !["Condición de carga de alerta de métrica"](media/action-groups-logic-app/metric-alert-payload-condition.png "Condición de carga de alerta de métrica")

  1. En la condición **if true**, agregue un bucle **For each** y la acción de Microsoft Teams. Defina el mensaje con una combinación de HTML y contenido dinámico.

      !["Acción posterior a la condición true de alerta de métrica"](media/action-groups-logic-app/metric-alert-true-condition-post-action.png "Acción posterior a la condición true de alerta de métrica")

  1. En la condición **If false**, defina una acción de Microsoft Teams que comunique que la alerta de métrica no coincide con las expectativas de la aplicación lógica. Incluya la carga de JSON. Observe cómo hacer referencia al contenido dinámico de `triggerBody` en la expresión `json()`.

      !["Acción posterior a la condición false de alerta de métrica"](media/action-groups-logic-app/metric-alert-false-condition-post-action.png "Acción posterior a la condición false de alerta de métrica")

- El paso 15 es igual. Siga las instrucciones para guardar la aplicación lógica y actualizar el grupo de acciones.

## <a name="calling-other-applications-besides-microsoft-teams"></a>Llamada a otras aplicaciones además de Microsoft Teams
Logic Apps tiene una serie de distintos conectores que permiten desencadenar acciones en una amplia gama de aplicaciones y bases de datos. Slack, SQL Server, Oracle, Salesforce, son solo algunos ejemplos. Para más información sobre los conectores, consulte [Conectores de Logic App](../../connectors/apis-list.md).  

## <a name="next-steps"></a>Pasos siguientes
* Consulte la [introducción a las alertas del registro de actividad de Azure](./alerts-overview.md) y aprenda cómo puede recibir alertas.  
* Aprenda a [configurar alertas siempre que se publique una notificación de Azure Service Health](../../service-health/alerts-activity-log-service-notifications-portal.md).
* Más información sobre los [grupos de acciones](./action-groups.md).
