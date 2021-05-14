---
title: Métricas de Azure Monitor para Application Gateway
description: Obtenga información sobre cómo usar métricas para supervisar el rendimiento de la puerta de enlace de aplicaciones
services: application-gateway
author: azhar2005
ms.service: application-gateway
ms.topic: article
ms.date: 04/19/2021
ms.author: azhussai
ms.openlocfilehash: 615db7e8d53e397755ae318d171dab1eab9ec6c6
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107727815"
---
# <a name="metrics-for-application-gateway"></a>Métricas para Application Gateway

Application Gateway publica puntos de datos, denominados métricas, para [Azure Monitor](../azure-monitor/overview.md) para el rendimiento de las instancias de Application Gateway y back-end. Estas métricas son valores numéricos de un conjunto ordenado de datos de serie temporal que describen algún aspecto de la puerta de enlace de aplicaciones en un momento determinado. Si hay solicitudes que fluyen a través de Application Gateway, mide y envía sus métricas en intervalos de 60 segundos. Si no hay ninguna solicitud que fluya a través de Application Gateway o no tiene datos para una métrica, no se informará de la métrica. Para obtener más información, vea [Información general sobre las métricas en Microsoft Azure](../azure-monitor/essentials/data-platform-metrics.md).

## <a name="metrics-supported-by-application-gateway-v2-sku"></a>Métricas compatibles con la SKU de Application Gateway V2

### <a name="timing-metrics"></a>Métricas de tiempo

Application Gateway proporciona varias métricas de temporización integradas relacionadas con la solicitud y la respuesta, que se miden en milisegundos. 

:::image type="content" source="./media/application-gateway-metrics/application-gateway-metrics.png" alt-text="[Diagrama de métricas de tiempo para Application Gateway" border="false":::

> [!NOTE]
>
> Si hay más de un agente de escucha en Application Gateway, filtre siempre por la dimensión *Agente de escucha* al comparar las diferentes métricas de latencia para obtener una inferencia significativa.

- **Tiempo de conexión de back-end**

  Tiempo empleado en establecer una conexión con la aplicación de back-end. 

  Esto incluye la latencia de red, así como el tiempo que tarda la pila TCP del servidor back-end en establecer nuevas conexiones. En el caso de TLS, incluye también el tiempo empleado en el protocolo de enlace. 

- **Tiempo de respuesta del primer byte de back-end**

  Intervalo de tiempo entre el inicio del establecimiento de una conexión con el servidor back-end y la recepción del primer byte del encabezado de la respuesta. 

  Se aproxima a la suma de *Tiempo de conexión de back-end*, el tiempo que tarda la solicitud en alcanzar el back-end desde Application Gateway, el tiempo que tarda la aplicación de back-end en responder (el tiempo que el servidor tardó en generar contenido, y posiblemente capturar las consultas de la base de datos) y el tiempo que tarda el primer byte de la respuesta en llegar a Application Gateway desde el back-end.

- **Tiempo de respuesta del último byte de back-end**

  Intervalo de tiempo entre el inicio del establecimiento de una conexión con el servidor back-end y la recepción del último byte del cuerpo de la respuesta. 

  Se aproxima a la suma de *Tiempo de respuesta del primer byte de back-end* y el tiempo de transferencia de datos (este número puede variar en gran medida según el tamaño de los objetos solicitados y la latencia de la red del servidor).

- **Tiempo total de Application Gateway**

  Promedio de tiempo que se tarda en recibir y procesar una solicitud y en enviar la respuesta. 

  Este es el intervalo desde el momento en que Application Gateway recibe el primer byte de la solicitud HTTP hasta el momento en que se envía el último byte de respuesta al cliente. Esto incluye el tiempo de procesamiento que tarda Application Gateway, el *Tiempo de respuesta del último byte de back-end*, el tiempo que tarda Application Gateway en enviar toda la respuesta y el *Cliente RTT*.

- **Cliente RTT**

  Tiempo medio de ida y vuelta entre clientes y Application Gateway.



Estas métricas se pueden usar para determinar si la ralentización observada se debe a la red del cliente, al rendimiento de Application Gateway, a la red de back-end y la saturación de la pila TCP del servidor back-end, al rendimiento de la aplicación de back-end o al tamaño de archivo grande.

Por ejemplo, si hay un pico en la tendencia de *Tiempo de respuesta del primer byte de back-end*, pero la tendencia de *Tiempo de conexión de back-end* es estable, se puede deducir que la latencia de Application Gateway al back-end y el tiempo necesario para establecer la conexión son estables, y el pico se debe a un aumento en el tiempo de respuesta de la aplicación de back-end. Por otro lado, si el pico en *Tiempo de respuesta del primer byte de back-end* se asocia a un pico correspondiente en *Tiempo de conexión de back-end*, se puede deducir que la red entre Application Gateway y el servidor back-end o la pila TCP del servidor back-end se han saturado. 

Si observa un pico en *Tiempo de respuesta del último byte de back-end*, pero el *Tiempo de respuesta del primer byte de back-end* es estable, se puede deducir que el pico se debe a que se está solicitando un archivo más grande.

Del mismo modo, si el *Tiempo total de Application Gateway* tiene un pico, pero el *Tiempo de respuesta del último byte de back-end* es estable, puede ser la señal de un cuello de botella de rendimiento en Application Gateway o de un cuello de botella en la red entre el cliente y Application Gateway. Además, si el *cliente RTT* también tiene un pico correspondiente, indica que la degradación se debe a la red entre el cliente y Application Gateway.

### <a name="application-gateway-metrics"></a>Métricas de Application Gateway

Para Application Gateway, están disponibles las métricas siguientes:

- **Bytes recibidos**

   Recuento de bytes recibidos por Application Gateway de los clientes

- **Bytes enviados**

   Recuento de bytes enviados por Application Gateway de los clientes

- **Protocolo TLS de cliente**

   Recuento de solicitudes TLS y no TLS iniciadas por el cliente que establecieron la conexión con Application Gateway. Para ver la distribución del protocolo TLS, filtre por el protocolo TLS de la dimensión.

- **Unidades de capacidad actuales**

   Recuento de unidades de capacidad consumidas para equilibrar la carga del tráfico. Hay tres determinantes para la unidad de capacidad: unidad de proceso, conexiones persistentes y rendimiento. Cada unidad de capacidad se compone a lo sumo de: 1 unidad de proceso, o 2500 conexiones persistentes, o rendimiento de 2,22 Mbps.

- **Unidades de proceso actuales**

   Recuento de la capacidad de procesador consumida. Los factores que afectan a la unidad de proceso son las conexiones TLS/seg, los cálculos de reescritura de direcciones URL y el procesamiento de reglas de WAF. 

- **Conexiones actuales**

   El número total de conexiones simultáneas activas de los clientes a Application Gateway.
   
- **Unidades de capacidad facturadas estimadas**

  Con el SKU de la versión 2, el modelo de precios está determinado por el consumo. Las unidades de capacidad miden el costo basado en el consumo, que se suma al costo fijo. Las *unidades de capacidad facturadas estimadas* indica el número de unidades de capacidad que se usa para estimar la facturación. Se calcula como el valor mayor entre las *unidades de capacidad actuales* (unidades de capacidad necesarias para equilibrar la carga del tráfico) y las *unidades de capacidad facturables fijas* (unidades de capacidad mínimas mantenidas).

- **Solicitudes con error**

  Número de solicitudes que ha servido Application Gateway con códigos de error del servidor 5xx. Esto incluye los códigos 5xx generados desde Application Gateway, así como los códigos 5xx que se generan desde el back-end. El recuento de solicitudes puede filtrarse aún más para mostrar el recuento por cada combinación de configuración de grupo de back-end o http específica.
   
- **Unidades de capacidad facturables fijas**

  El número mínimo de unidades de capacidad que se mantienen aprovisionadas según la configuración de *Unidades de escalado mínimas* (una instancia se traduce en 10 unidades de capacidad) en la configuración de Application Gateway.
   
 - **Nuevas conexiones por segundo**

   Número promedio de conexiones TCP nuevas por segundo establecidas desde los clientes con Application Gateway y desde Application Gateway con los miembros de back-end.


- **Estado de respuesta**

   Estado de la respuesta HTTP devuelta por Application Gateway. La distribución del código de estado de respuesta se puede categorizar aún más para mostrar las respuestas en categorías 2xx, 3xx, 4xx y 5xx.

- **Rendimiento**

   Número de bytes por segundo que ha ofrecido Application Gateway

- **Total de solicitudes**

   Recuento de solicitudes correctas que ha servido Application Gateway. El recuento de solicitudes puede filtrarse aún más para mostrar el recuento por cada combinación de configuración de grupo de back-end o http específica.

### <a name="backend-metrics"></a>Métricas de back-end

Para Application Gateway, están disponibles las métricas siguientes:

- **Estado de respuesta de back-end**

  Recuento de códigos de estado de respuesta HTTP devueltos por los back-ends. No se incluyen los códigos de respuesta generados por Application Gateway. La distribución del código de estado de respuesta se puede categorizar aún más para mostrar las respuestas en categorías 2xx, 3xx, 4xx y 5xx.

- **Recuento de hosts con estado correcto**

  El número de back-ends que el sondeo de Estado ha determinado que son correctos. También puede filtrar en función de grupos de back-end para mostrar el número de hosts en buen estado en un grupo de back-end específico.

- **Recuento de hosts con estado incorrecto**

  El número de back-ends que el sondeo de Estado ha determinado que son incorrectos. También puede filtrar en función de grupos de back-end para mostrar el número de hosts en estado incorrecto en un grupo de back-end específico.
  
- **Solicitudes por minuto y host con estado correcto**

  Número promedio de solicitudes recibidas por cada miembro correcto en un grupo de servidores de back-end en un minuto. Debe especificar el grupo de servidores de back-end mediante la dimensión *BackendPool Httpsettings*.  
  

## <a name="metrics-supported-by-application-gateway-v1-sku"></a>Métricas compatibles con la SKU de Application Gateway V1

### <a name="application-gateway-metrics"></a>Métricas de Application Gateway

Para Application Gateway, están disponibles las métricas siguientes:

- **Uso de CPU**

  Muestra el uso de las CPU asignadas a Application Gateway.  En condiciones normales, el uso de CPU no debe superar con regularidad el 90 %, ya que esto puede provocar latencia en los sitios web hospedados tras Application Gateway e interrumpir la experiencia del cliente. Para controlar o mejorar indirectamente el uso de CPU, puede modificar la configuración de Application Gateway aumentando el número de instancias o pasando a un tamaño de SKU mayor, o bien haciendo ambas cosas.

- **Conexiones actuales**

  Recuento de conexiones actuales establecidas con Application Gateway

- **Solicitudes con error**

  Número de solicitudes erróneas debido a problemas de conexión. Este recuento incluye las solicitudes que devolvieron algún error debido a que se superó el "tiempo de espera de la solicitud" de la configuración HTTP y las solicitudes que devolvieron algún error debido a problemas de conexión entre la puerta de enlace de aplicaciones y el back-end. Sin embargo, este recuento no incluye los errores que se hayan producido debido a que no hay ningún back-end correcto disponible. las respuestas 4xx y 5xx del back-end tampoco se consideran como parte de esta métrica.

- **Estado de respuesta**

  Estado de la respuesta HTTP devuelta por Application Gateway. La distribución del código de estado de respuesta se puede categorizar aún más para mostrar las respuestas en categorías 2xx, 3xx, 4xx y 5xx.

- **Rendimiento**

  Número de bytes por segundo que ha ofrecido Application Gateway

- **Total de solicitudes**

  Recuento de solicitudes correctas que ha servido Application Gateway. El recuento de solicitudes puede filtrarse aún más para mostrar el recuento por cada combinación de configuración de grupo de back-end o http específica.

- **Web Application Firewall Blocked Requests Count** (Recuento de solicitudes bloqueadas por el firewall de aplicaciones web)
- **Web Application Firewall Blocked Requests Rule Distribution** (Distribución de reglas de solicitudes bloqueadas por el firewall de aplicaciones web)
- **Web Application Firewall Total Rule Distribution** (Distribución de reglas totales del firewall de aplicaciones web)

### <a name="backend-metrics"></a>Métricas de back-end

Para Application Gateway, están disponibles las métricas siguientes:

- **Recuento de hosts con estado correcto**

  El número de back-ends que el sondeo de Estado ha determinado que son correctos. También puede filtrar en función de grupos de back-end para mostrar el número de hosts en buen estado en un grupo de back-end específico.

- **Recuento de hosts con estado incorrecto**

  El número de back-ends que el sondeo de Estado ha determinado que son incorrectos. También puede filtrar en función de grupos de back-end para mostrar el número de hosts en estado incorrecto en un grupo de back-end específico.

## <a name="metrics-visualization"></a>Visualización de las métricas

Navegue a una instancia de Application Gateway y, en **Supervisión**, seleccione **Métricas**. Para ver los valores disponibles, seleccione la lista desplegable **MÉTRICA**.

En la siguiente imagen, verá un ejemplo con tres métricas que se muestran para los últimos 30 minutos:

:::image type="content" source="media/application-gateway-diagnostics/figure5.png" alt-text="Vista de métricas." lightbox="media/application-gateway-diagnostics/figure5-lb.png":::

Para ver una lista de métricas actuales, consulte el artículo de [métricas compatibles con Azure Monitor](../azure-monitor/essentials/metrics-supported.md).

### <a name="alert-rules-on-metrics"></a>Reglas de alerta en métricas

Puede iniciar las reglas de alerta en función de las métricas de un recurso. Por ejemplo, una alerta puede llamar a un webhook o enviar un correo electrónico a un administrador si el rendimiento de la puerta de enlace de aplicaciones es superior, igual o inferior a un umbral durante un período especificado.

En el ejemplo siguiente, se explica paso a paso cómo crear una regla de alerta que envía un correo electrónico a un administrador cuando el rendimiento supera un umbral:

1. Seleccione **Agregar alerta de métrica** para abrir la hoja **Agregar regla**. También puede acceder a esta página desde la página de métricas.

   ![Botón “Agregar alerta de métrica”][6]

2. En la página **Agregar regla**, rellene las secciones de nombre, condición y notificación, y seleccione **Aceptar**.

   * En el selector **Condición**, seleccione uno de los cuatro valores: **Mayor que**, **Mayor o igual que**, **Menor que** o **Menor o igual que**.

   * En el selector **Período**, seleccione un período de cinco minutos a seis horas.

   * Al seleccionar **Lectores, colaboradores y propietarios de correo electrónico**, el correo electrónico puede ser dinámico según los usuarios que tengan acceso a ese recurso. De lo contrario, puede proporcionar una lista separada por comas de los usuarios en el cuadro de texto **Correos electrónicos de administrador adicionales**.

   ![Página Agregar regla][7]

Si se supera el umbral, llegará un correo electrónico similar al que se muestra en la siguiente imagen:

![Correo electrónico para el umbral infringido][8]

Después de crear una alerta de métrica, aparece una lista de alertas. Proporciona una visión general de todas las reglas de alerta.

![Lista de alertas y reglas][9]

Para más información acerca de las notificaciones de alerta, consulte [Recibir notificaciones de alerta](../azure-monitor/alerts/alerts-overview.md).

Para conocer más detalles sobre los webhooks y cómo usarlos con las alertas, visite [Configuración de un webhook en una alerta de métrica de Azure](../azure-monitor/alerts/alerts-webhooks.md).

## <a name="next-steps"></a>Pasos siguientes

* Visualice el contador y los registros de eventos mediante los [registros de Azure Monitor](../azure-monitor/insights/azure-networking-analytics.md).
* Consulte la entrada de blog [Visualize your Azure Activity Logs with Power BI](https://powerbi.microsoft.com/blog/monitor-azure-audit-logs-with-power-bi/) (Visualizar los registros de actividades de Azure con Power BI).
* Consulte la entrada de blog [View and analyze Azure Audit Logs in Power BI and more](https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/) (Consulta y análisis de registros de auditoría de Azure en Power BI y más).

[1]: ./media/application-gateway-diagnostics/figure1.png
[2]: ./media/application-gateway-diagnostics/figure2.png
[3]: ./media/application-gateway-diagnostics/figure3.png
[4]: ./media/application-gateway-diagnostics/figure4.png
[5]: ./media/application-gateway-diagnostics/figure5.png
[6]: ./media/application-gateway-diagnostics/figure6.png
[7]: ./media/application-gateway-diagnostics/figure7.png
[8]: ./media/application-gateway-diagnostics/figure8.png
[9]: ./media/application-gateway-diagnostics/figure9.png
[10]: ./media/application-gateway-diagnostics/figure10.png
