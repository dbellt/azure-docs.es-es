---
title: Solución de problemas con los registros de diagnóstico del servicio Azure Web PubSub
description: Aprenda a obtener registros de diagnóstico y usarlos para solucionar problemas.
author: yjin81
ms.author: yajin1
ms.service: azure-web-pubsub
ms.topic: how-to
ms.date: 03/22/2021
ms.openlocfilehash: 119591c4a337cf622b95fa4ed70a7508d8acb99e
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111963089"
---
# <a name="how-to-troubleshoot-with-diagnostic-logs"></a>Solución de problemas con registros de diagnóstico

En esta guía paso a paso se muestran las opciones para obtener los registros de diagnóstico y cómo usarlos para solucionar problemas.

## <a name="whats-the-diagnostic-logs"></a>¿Qué son los registros de diagnóstico?

Los registros de diagnóstico proporcionan una vista más completa de la conectividad y la información de mensajería a la instancia del servicio Azure Web PubSub. Se pueden usar para la identificación de problemas, el seguimiento de la conexión, el seguimiento de los mensajes y el análisis.

Hay dos tipos de registros: el registro de conectividad y el registro de mensajería.

### <a name="connectivity-logs"></a>Registros de conectividad

Los registros de conectividad proporcionan información detallada para las conexiones del centro de conectividad de Azure Web PubSub. Por ejemplo, información básica (identificador de usuario, identificador de conexión, etc.) e información de eventos (evento de conexión, desconexión y anulación, etc.). Es por esto que el registro de conectividad resulta útil para solucionar problemas relacionados con la conexión. 

### <a name="messaging-logs"></a>Registros de mensajería

Los registros de mensajería proporcionan información de seguimiento de los mensajes del centro de conectividad de Azure Web PubSub recibidos y enviados a través del servicio Azure Web PubSub. Por ejemplo, el identificador de seguimiento y el tipo de mensaje del mensaje. Por lo general, el mensaje se registra cuando llega al servidor o cuando sale de este. Por lo tanto, los registros de mensajería resultan útiles para solucionar problemas relacionados con los mensajes. 

## <a name="capture-diagnostic-logs-with-azure-web-pubsub-service-live-trace-tool"></a>Captura de registros de diagnóstico con la herramienta de seguimiento activo del servicio Azure Web PubSub 

La herramienta de seguimiento activo del servicio Azure Web PubSub tiene la capacidad de recopilar registros de diagnóstico en tiempo real y es útil para realizar un seguimiento con el entorno de desarrollo del cliente. La herramienta de seguimiento activo podría capturar tanto registros de conectividad como registros de mensajería.

> [!NOTE]
> Los registros de diagnóstico en tiempo real que captura la herramienta de seguimiento activo se facturarán como mensajes (tráfico saliente).

> [!NOTE]
> La instancia del servicio Azure Web PubSub creada como nivel gratis tiene el límite de mensajes diario (tráfico saliente).

### <a name="launch-the-live-trace-tool"></a>Inicio de la herramienta de seguimiento activo

1. Vaya a Azure Portal. 
1. En la página **Configuración de diagnóstico** de la instancia del servicio Azure Web PubSub, seleccione **Open Live Trace Tool** (Abrir herramienta de seguimiento activo). 

    :::image type="content" source="./media/howto-troubleshoot-diagnostic-logs/diagnostic-logs-with-live-trace-tool.png" alt-text="Inicio de la herramienta de seguimiento activo.":::

### <a name="capture-the-diagnostic-logs"></a>Captura de los registros de diagnóstico

La herramienta de seguimiento activo proporciona algunas funcionalidades esenciales que lo ayudarán a capturar los registros de diagnóstico para solucionar problemas.

* **Capture** (Capturar): empiece a capturar los registros de diagnóstico en tiempo real de la instancia de Azure Web PubSub con la herramienta de seguimiento activo.
* **Clear** (Borrar): borre los registros de diagnóstico capturados en tiempo real.
* **Log filter** (Filtro de registro): la herramienta de seguimiento activo permite filtrar los registros de diagnóstico capturados en tiempo real con una palabra clave específica. El separador común (por ejemplo, espacio, coma, punto y coma, etc.) se considerará parte de la palabra clave. 
* **Status** (Estado): el estado muestra si la herramienta de seguimiento activo está conectada o desconectada de la instancia específica.

:::image type="content" source="./media/howto-troubleshoot-diagnostic-logs/live-trace-tool-capture.png" alt-text="Captura de registros de diagnóstico con la herramienta de seguimiento activo.":::

Los registros de diagnóstico en tiempo real que captura la herramienta de seguimiento activo contienen información detallada para la solución de problemas. 

| Nombre | Descripción |
| ------------ |  ------------------------ | 
| Time | Hora del evento de registro |
| Nivel de registro | Nivel del evento de registro (seguimiento/depuración/informativo/advertencia/error) |
| Nombre del evento | Nombre de operación del evento |
| Message | Mensaje detallado del evento de registro |
| Excepción | Excepción en tiempo de ejecución del servicio Azure Web PubSub |
| Hub | Nombre del centro de conectividad definido por el usuario |
| Id. de conexión | Identidad de la conexión |
| Id. de usuario | Identidad del usuario |
| IP | Dirección IP del cliente | 

Una vez que el servicio Azure Web PubSub esté disponible de manera general, la herramienta de seguimiento activo también admitirá la exportación de los registros como un formato específico y, luego, lo ayudará a compartir con otros usuarios para solucionar problemas. 

## <a name="capture-diagnostic-logs-with-azure-monitor"></a>Captura de registros de diagnóstico con Azure Monitor

La versión preliminar pública no admite la integración con [Azure Monitor](https://azure.microsoft.com/services/monitor/), [Azure Storage](../azure-monitor/essentials/resource-logs.md#send-to-azure-storage) y [Log Analytics](../azure-monitor/essentials/resource-logs.md#send-to-log-analytics-workspace) para capturar los registros de diagnóstico.

## <a name="troubleshoot-with-the-diagnostic-logs"></a>Solución de problemas con los registros de diagnóstico

Cuando se encuentra en una situación inesperada de crecimiento o interrupción de la conexión, puede aprovechar los registros de diagnóstico para solucionar problemas. Los problemas típicos suelen ser cambios inesperados en el número de conexiones, conexiones que llegan a los límites de conexión y errores de autorización.

### <a name="unexpected-connection-number-changes"></a>Cambios inesperados en el número de conexiones

#### <a name="unexpected-connection-dropping"></a>Interrupción inesperada de la conexión

Si se interrumpe una conexión, los registros de diagnóstico registrarán este evento de desconexión con `ConnectionAborted` o `ConnectionEnded` en `operationName`.

La diferencia entre `ConnectionAborted` y `ConnectionEnded` es que `ConnectionEnded` es una desconexión esperada desencadenada por el lado cliente o servidor. Aunque `ConnectionAborted` suele ser un evento de interrupción inesperada de la conexión, se proporcionará el motivo de la anulación en `message`.

Las opciones de la anulación se muestran en la siguiente tabla:

| Motivo | Descripción |
| ------- | ------- |
| El número de conexiones llega al límite | El número de conexiones llega al límite del nivel de precios actual. Considere la posibilidad de escalar la unidad de servicio.
| Recarga de servicios, reconexión | El servicio Azure Web PubSub se está recargando. Debe implementar su propio mecanismo de reconexión o volver a conectarse manualmente al servicio Azure Web PubSub. |
| Error transitorio del servidor interno | Se produce un error transitorio en el servicio Azure Web PubSub, se debería recuperar automáticamente.

#### <a name="unexpected-connection-growing"></a>Crecimiento de conexiones inesperado

Para solucionar los problemas relacionados con un crecimiento de conexiones inesperado, lo primero que debe hacer es filtrar las conexiones adicionales. Puede agregar un identificador único de usuario de prueba a la conexión del cliente de prueba. A continuación, compruébelo con los registros de diagnóstico; si ve que hay más de una conexión de cliente con el mismo identificador de usuario o IP de prueba, es probable que el lado cliente cree y establezca más conexiones de las esperadas. Compruebe el lado cliente.

### <a name="authorization-failure"></a>Error de autorización

Si recibe de vuelta 401 No autorizado para las solicitudes de cliente, compruebe los registros de diagnóstico. Si encuentra `Failed to validate audience. Expected Audiences: <valid audience>. Actual Audiences: <actual audience>`, significa que no hay ninguna audiencia del token de acceso válida. Intente usar las audiencias válidas sugeridas en el registro.

### <a name="throttling"></a>Limitaciones

Si encuentra que no puede establecer conexiones de cliente con el servicio Azure Web PubSub, compruebe los registros de diagnóstico. Si encuentra `Connection count reaches limit` en el registro de diagnóstico, es porque establece demasiadas conexiones con el servicio Azure Web PubSub, lo que llega al límite del número de conexiones. Considere la posibilidad de escalar verticalmente la instancia del servicio Azure Web PubSub. Si encuentra `Message count reaches limit` en el registro de diagnóstico, significa que utiliza el nivel gratuito y ha llegado al máximo de la cuota de mensajes. Si desea enviar más mensajes, considere la posibilidad de cambiar la instancia del servicio Azure Web PubSub al nivel estándar para enviar mensajes adicionales. Para más información, consulte [Precios de Azure Web PubSub](https://azure.microsoft.com/pricing/details/web-pubsub/).