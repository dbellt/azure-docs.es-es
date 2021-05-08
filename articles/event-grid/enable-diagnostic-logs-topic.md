---
title: 'Azure Event Grid: Habilitación de los registros de diagnóstico de temas o dominios'
description: En este artículo se proporcionan instrucciones paso a paso sobre cómo habilitar los registros de diagnóstico para un tema de Azure Event Grid.
ms.topic: how-to
ms.date: 04/22/2021
ms.openlocfilehash: 78dfeed0cedfe96d9a0d70411aecc7c7f2c51a72
ms.sourcegitcommit: 19dcad80aa7df4d288d40dc28cb0a5157b401ac4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107897782"
---
#  <a name="enable-diagnostic-logs-for-azure-event-grid-topics-or-domains"></a>Habilitación de registros de diagnóstico para los temas o dominios de Azure Event Grid
En este artículo, se proporcionan instrucciones paso a paso acerca de cómo habilitar los registros de diagnóstico en un dominio o tema de Event Grid.  Esta configuración permite capturar y ver los registros de **errores de publicación y entrega**. 

> [!IMPORTANT]
> Para el esquema de los registros de diagnóstico, vea [Registros de diagnóstico](diagnostic-logs.md). 


## <a name="prerequisites"></a>Prerrequisitos

- Un tema de Event Grid aprovisionado.
- Un destino aprovisionado para capturar registros de diagnóstico. Puede ser uno de los siguientes destinos que se encuentran en la misma ubicación que el tema de Event Grid:
    - Cuenta de almacenamiento de Azure
    - Centro de eventos
    - Área de trabajo de Log Analytics

## <a name="enable-diagnostic-logs-for-a-custom-topic"></a>Habilitación de los registros de diagnóstico de un tema personalizado

> [!NOTE]
> El siguiente procedimiento proporciona instrucciones paso a paso para habilitar los registros de diagnóstico para un tema. Los pasos para habilitar los registros de diagnóstico para un dominio son muy similares. En el paso 2, vaya al **dominio** de Event Grid en Azure Portal.  

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Vaya al tema de Event Grid para el que desee habilitar la configuración del registro de diagnóstico. 
    1. En la barra de búsqueda de la parte superior, busque **Temas de Event Grid**. 
    
        ![Buscar temas personalizados](./media/enable-diagnostic-logs-topic/search-custom-topics.png)
    1. Seleccione el **tema** de la lista para el que desea configurar las opciones de diagnóstico. 
1. Seleccione **Configuración de diagnóstico** en **Supervisión**, en el menú de la izquierda.
1. En la página **Configuración de diagnóstico**, seleccione **Add New Diagnostic Setting** (Agregar nueva configuración de diagnóstico). 
    
    ![Botón Agregar configuración de diagnóstico](./media/enable-diagnostic-logs-topic/diagnostic-settings-add.png)
5. Especifique un **nombre** para la configuración de diagnóstico. 
6. Seleccione las opciones **DeliveryFailures** y **PublishFailures** en la sección **Registro**. 
    ![Seleccione los errores](./media/enable-diagnostic-logs-topic/log-failures.png)
7. Habilite uno o varios destinos de captura para los registros y, a continuación, configúrelos; para ello, seleccione un recurso de captura creado anteriormente. 
    - Si selecciona **Archive to a storage account** (Archivar en una cuenta de almacenamiento), seleccione **Cuenta de almacenamiento: configurar** y, a continuación, seleccione la cuenta de almacenamiento en la suscripción de Azure. 

        ![Captura de pantalla que muestra la página "Configuración de diagnóstico" con la opción "Archive to an Azure storage account" (Archivar en una cuenta de almacenamiento de Azure) y una cuenta de almacenamiento seleccionada.](./media/enable-diagnostic-logs-topic/archive-storage.png)
    - Si selecciona **Stream to an event hub** (Transmitir a un centro de eventos), seleccione **Centro de eventos: Configurar** y, a continuación, seleccione el espacio de nombres de Event Hubs, el centro de eventos y la directiva de acceso. 
        ![Captura de pantalla que muestra la página "Configuración de diagnóstico" con la opción "Transmitir a un centro de eventos" activada.](./media/enable-diagnostic-logs-topic/archive-event-hub.png)
    - Si selecciona **Send to Log Analytics** (Enviar a Log Analytics), seleccione un área de trabajo de Log Analytics.
        ![Captura de pantalla que muestra la página "Configuración de diagnóstico" con la opción "Enviar a Log Analytics" activada.](./media/enable-diagnostic-logs-topic/send-log-analytics.png)
8. Seleccione **Guardar**. Después, Seleccione **X** en la esquina derecha para cerrar la página. 
9. Ahora, vuelva a la página **Configuración de diagnóstico**, confirme que ve una nueva entrada en la tabla **Configuración de diagnóstico**. 
    ![Captura de pantalla que muestra la página "Configuración de diagnóstico" con una nueva entrada resaltada en la tabla "Configuración de diagnóstico".](./media/enable-diagnostic-logs-topic/diagnostic-setting-list.png)

     También puede habilitar la recopilación de todas las métricas del tema. 

## <a name="enable-diagnostic-logs-for-a-system-topic"></a>Habilitación de los registros de diagnóstico de un tema de sistema

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Vaya al tema de Event Grid para el que desee habilitar la configuración del registro de diagnóstico. 
    1. En la barra de búsqueda de la parte superior, busque **Temas del sistema de Event Grid**. 
    
        ![Búsqueda de temas del sistema](./media/enable-diagnostic-logs-topic/search-system-topics.png)
    1. Seleccione el **tema del sistema** para el que desea configurar las opciones de diagnóstico. 
    
        ![Seleccionar el tema del sistema](./media/enable-diagnostic-logs-topic/select-system-topic.png)
3. Seleccione **Configuración de diagnóstico** en **Supervisión** en el menú izquierdo y, luego, elija **Agregar configuración de diagnóstico**. 

    ![Agregar configuración de diagnóstico: botón](./media/enable-diagnostic-logs-topic/system-topic-add-diagnostic-settings-button.png)
4. Especifique un **nombre** para la configuración de diagnóstico. 
7. Seleccione **DeliveryFailures** en la sección de **Registro**. 
    ![Seleccionar errores de entrega](./media/enable-diagnostic-logs-topic/system-topic-select-delivery-failures.png)
6. Habilite uno o varios destinos de captura para los registros y, a continuación, configúrelos; para ello, seleccione un recurso de captura creado anteriormente. 
    - Si selecciona **Send to Log Analytics** (Enviar a Log Analytics), seleccione un área de trabajo de Log Analytics.
        ![Enviar a Log Analytics](./media/enable-diagnostic-logs-topic/system-topic-select-log-workspace.png) 
    - Si selecciona **Archive to a storage account** (Archivar en una cuenta de almacenamiento), seleccione **Cuenta de almacenamiento: configurar** y, a continuación, seleccione la cuenta de almacenamiento en la suscripción de Azure. 

        ![Archivo en una cuenta de almacenamiento de Azure](./media/enable-diagnostic-logs-topic/system-topic-select-storage-account.png)
    - Si selecciona **Stream to an event hub** (Transmitir a un centro de eventos), seleccione **Centro de eventos: Configurar** y, a continuación, seleccione el espacio de nombres de Event Hubs, el centro de eventos y la directiva de acceso. 
        ![Transmisión a un centro de eventos](./media/enable-diagnostic-logs-topic/system-topic-select-event-hub.png)
8. Seleccione **Guardar**. Después, Seleccione **X** en la esquina derecha para cerrar la página. 
9. Ahora, vuelva a la página **Configuración de diagnóstico**, confirme que ve una nueva entrada en la tabla **Configuración de diagnóstico**. 
    ![Configuración de diagnóstico en la lista](./media/enable-diagnostic-logs-topic/system-topic-diagnostic-settings-targets.png)

     También puede habilitar la recopilación de todas las **métricas** del tema del sistema.

    ![Tema del sistema: habilitar todas las métricas](./media/enable-diagnostic-logs-topic/system-topics-metrics.png)

## <a name="view-diagnostic-logs-in-azure-storage"></a>Visualización de registros de diagnóstico en Azure Storage 

1. Una vez que se ha habilitado una cuenta de almacenamiento como destino de captura, Event Grid comienza a emitir registros de diagnóstico. Debería ver dos nuevos contenedores, **insights-logs-deliveryfailures** e **insights-logs-publishfailures**, en la cuenta de almacenamiento. 

    ![Almacenamiento: contenedores para registros de diagnóstico](./media/enable-diagnostic-logs-topic/storage-containers.png)
2. Cuando se desplace por uno de los contenedores, terminará en un blob en formato JSON. El archivo contiene entradas de registro para un error de entrega o un error de publicación. La ruta de navegación representa el **ResourceId** del tema de Event Grid y la marca de tiempo (nivel de minuto) como cuando se emitieron las entradas de registro. El archivo blob/JSON, que se puede descargar, al final se adhiere al esquema descrito en la sección siguiente. 

    [ ![Archivo JSON en el almacenamiento](./media/enable-diagnostic-logs-topic/select-json.png) ](./media/enable-diagnostic-logs-topic/select-json.png)
3. Debería ver el contenido en el archivo JSON similar al ejemplo siguiente: 

    ```json
    {
        "time": "2019-11-01T00:17:13.4389048Z",
        "resourceId": "/SUBSCRIPTIONS/SAMPLE-SUBSCTIPTION-ID /RESOURCEGROUPS/SAMPLE-RESOURCEGROUP-NAME/PROVIDERS/MICROSOFT.EVENTGRID/TOPICS/SAMPLE-TOPIC-NAME ",
        "eventSubscriptionName": "SAMPLEDESTINATION",
        "category": "DeliveryFailures",
        "operationName": "Deliver",
        "message": "Message:outcome=NotFound, latencyInMs=2635, id=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxx, systemId=xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx, state=FilteredFailingDelivery, deliveryTime=11/1/2019 12:17:10 AM, deliveryCount=0, probationCount=0, deliverySchema=EventGridEvent, eventSubscriptionDeliverySchema=EventGridEvent, fields=InputEvent, EventSubscriptionId, DeliveryTime, State, Id, DeliverySchema, LastDeliveryAttemptTime, SystemId, fieldCount=, requestExpiration=1/1/0001 12:00:00 AM, delivered=False publishTime=11/1/2019 12:17:10 AM, eventTime=11/1/2019 12:17:09 AM, eventType=Type, deliveryTime=11/1/2019 12:17:10 AM, filteringState=FilteredWithRpc, inputSchema=EventGridEvent, publisher=DIAGNOSTICLOGSTEST-EASTUS.EASTUS-1.EVENTGRID.AZURE.NET, size=363, fields=Id, PublishTime, SerializedBody, EventType, Topic, Subject, FilteringHashCode, SystemId, Publisher, FilteringTopic, TopicCategory, DataVersion, MetadataVersion, InputSchema, EventTime, fieldCount=15, url=sb://diagnosticlogstesting-eastus.servicebus.windows.net/, deliveryResponse=NotFound: The messaging entity 'sb://diagnosticlogstesting-eastus.servicebus.windows.net/eh-diagnosticlogstest' could not be found. TrackingId:c98c5af6-11f0-400b-8f56-c605662fb849_G14, SystemTracker:diagnosticlogstesting-eastus.servicebus.windows.net:eh-diagnosticlogstest, Timestamp:2019-11-01T00:17:13, referenceId: ac141738a9a54451b12b4cc31a10dedc_G14:"
    }
    ```
## <a name="next-steps"></a>Pasos siguientes
Para obtener el esquema de registro y otra información conceptual sobre los registros de diagnóstico de temas o dominios, consulte [Registros de diagnóstico](diagnostic-logs.md).
