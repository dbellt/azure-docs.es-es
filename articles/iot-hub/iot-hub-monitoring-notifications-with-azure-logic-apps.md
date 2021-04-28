---
title: Supervisión remota y notificaciones de IoT con Azure Logic Apps | Microsoft Docs
description: Use Azure Logic Apps para la supervisión de temperatura de IoT en IoT Hub y el envío automático de notificaciones de correo electrónico al buzón de correo cada vez que se detecten anomalías.
author: robinsh
keywords: supervisión de iot, notificaciones de iot, supervisión de temperatura de iot
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 07/18/2019
ms.author: robinsh
ms.openlocfilehash: 74724357dea9cd6c8c89a11a9eeb3d1b2933b790
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107564953"
---
# <a name="iot-remote-monitoring-and-notifications-with-azure-logic-apps-connecting-your-iot-hub-and-mailbox"></a>Supervisión remota y notificaciones de IoT con Azure Logic Apps conectando IoT Hub y el buzón de correo

![Diagrama integral](media/iot-hub-monitoring-notifications-with-azure-logic-apps/iot-hub-e2e-logic-apps.png)

[!INCLUDE [iot-hub-get-started-note](../../includes/iot-hub-get-started-note.md)]

[Azure Logic Apps](../logic-apps/index.yml) puede ayudar a orquestar flujos de trabajo entre servicios locales y de nube, una o más empresas y entre distintos protocolos. Una aplicación lógica comienza con un desencadenador, seguido de una o varias acciones que se pueden secuenciar mediante controles integrados, como condiciones e iteradores. Esta flexibilidad hace que Logic Apps sea una solución de IoT ideal para los escenarios de supervisión de IoT. Por ejemplo, la llegada de los datos de telemetría desde un dispositivo en un punto de conexión de IoT Hub puede iniciar flujos de trabajo de aplicaciones lógicas para almacenar los datos en un blob de Azure Storage, enviar alertas por correo electrónico para avisar de anomalías de datos, programar una visita del técnico si un dispositivo notifica un error, etc.

En este artículo aprenderá a crear una aplicación lógica que conecte IoT Hub y el buzón de correo para la supervisión de temperatura y las notificaciones. El código de cliente que se ejecuta en el dispositivo establece una propiedad de aplicación, `temperatureAlert`, en cada mensaje de telemetría que le envía al centro de IoT. Cuando el código de cliente detecta una temperatura superior a 30 C, establece esta propiedad en `true`; en caso contrario, la establece en `false`.

Los mensajes que llegan a IoT Hub son similares al siguiente, con los datos de telemetría en el cuerpo y la propiedad `temperatureAlert` en las propiedades de la aplicación (no se muestran las propiedades del sistema):

```json
{
  "body": {
    "messageId": 18,
    "deviceId": "Raspberry Pi Web Client",
    "temperature": 27.796111770668457,
    "humidity": 66.77637926438427
  },
  "applicationProperties": {
    "temperatureAlert": "false"
  }
}
```

Para más información acerca del formato de los mensajes de IoT Hub, consulte [Creación y lectura de mensajes de IoT Hub](iot-hub-devguide-messages-construct.md).

En este tema, se configura el enrutamiento en IoT Hub para enviar mensajes en los que la propiedad `temperatureAlert` es `true` en un punto de conexión de Service Bus. Después, se configura una aplicación lógica que se desencadena en los mensajes que llegan al punto de conexión de Service Bus y le envía una notificación por correo electrónico.

## <a name="prerequisites"></a>Requisitos previos

* Realice el tutorial del [simulador en línea de Raspberry Pi](iot-hub-raspberry-pi-web-simulator-get-started.md), o bien uno de los tutoriales del dispositivo. Por ejemplo, puede ir a [Raspberry Pi con node.js](iot-hub-raspberry-pi-kit-node-get-started.md) o a uno de los inicios rápidos de [envío de telemetría](quickstart-send-telemetry-dotnet.md). Estos artículos abarcan los requisitos siguientes:

  * Una suscripción de Azure activa.
  * Un centro de Azure IoT en su suscripción.
  * Una aplicación cliente que se ejecuta en el dispositivo que envía mensajes de telemetría a Azure IoT Hub.

## <a name="create-service-bus-namespace-and-queue"></a>Crear un espacio de nombres de Service Bus y una cola

Cree un espacio de nombres de Service Bus y una cola. Más adelante en este tema, creará una regla de enrutamiento en el centro de IoT para dirigir los mensajes que contienen una alerta de temperatura a la cola de Service Bus, donde una aplicación lógica lo recogerá y activará para enviar una notificación por correo electrónico.

### <a name="create-a-service-bus-namespace"></a>Creación de un espacio de nombres de Service Bus

1. En [Azure Portal](https://portal.azure.com/), seleccione **+Crear un recurso** > **Integración** > **Service Bus**.

1. En el panel **Crear espacio de nombres**, proporcione la siguiente información:

   **Name**: el nombre del espacio de nombres de Service Bus. El nombre debe ser único en todo Azure.

   **Plan de tarifa**: Seleccione **Básica** de la lista desplegable. El nivel Básico es suficiente para este tutorial.

   **Grupo de recursos**: use el mismo grupo de recursos que usa el centro de IoT.

   **Ubicación**: use la misma ubicación que emplea el centro de IoT.

   ![Crear un espacio de nombres de Service Bus en Azure Portal](media/iot-hub-monitoring-notifications-with-azure-logic-apps/1-create-service-bus-namespace-azure-portal.png)

1. Seleccione **Crear**. Espere a que la implementación se complete antes de continuar con el paso siguiente.

### <a name="add-a-service-bus-queue-to-the-namespace"></a>Agregar una cola de Service Bus al espacio de nombres

1. Abra el espacio de nombre de Service Bus. La manera más fácil abrir el espacio de nombres de Service Bus es seleccionar **Grupos de recursos** del panel recursos, seleccionar el grupo de recursos y luego seleccionar el espacio de nombres de Service Bus de la lista de recursos.

1. En el panel **Espacio de nombres de Service Bus**, seleccione **+ Cola**.

1. Escriba un nombre para la cola y haga clic en **Crear**. Cuando la cola se haya creado correctamente, el panel **Crear cola** se cierra.

   ![Agregar una cola de Service Bus en Azure Portal](media/iot-hub-monitoring-notifications-with-azure-logic-apps/create-service-bus-queue.png)

1. En el panel **Espacio de nombres de Service Bus**, en **Entidades**, seleccione **Colas**. Abra la cola de Service Bus de la lista y seleccione **Directivas de acceso compartido** >  **+ Agregar**.

1. Escriba un nombre para la directiva, active **Administrar** y luego haga clic en **Crear**.

   ![Agregar una cola de Service Bus en Azure Portal](media/iot-hub-monitoring-notifications-with-azure-logic-apps/2-add-service-bus-queue-azure-portal.png)

## <a name="add-a-custom-endpoint-and-routing-rule-to-your-iot-hub"></a>Agregar un punto de conexión personalizado y una regla de enrutamiento al centro de IoT

Agregue un punto de conexión personalizado para la cola de Service Bus al centro de IoT y cree una regla de enrutamiento de mensajes para dirigir los mensajes que contienen una alerta de temperatura a ese punto de conexión, donde la aplicación lógica los recogerá. La regla de enrutamiento usa una consulta de enrutamiento, `temperatureAlert = "true"`, para reenviar los mensajes según el valor de la propiedad de aplicación `temperatureAlert` que establece el código de cliente que se ejecuta en el dispositivo. Para más información, consulte [Consulta de enrutamiento de mensajes basada en las propiedades del mensaje](./iot-hub-devguide-routing-query-syntax.md#message-routing-query-based-on-message-properties).

### <a name="add-a-custom-endpoint"></a>Agregar un punto de conexión personalizado

1. Abra el IoT Hub. La manera más fácil llegar al centro de IoT es seleccionar **Grupos de recursos** en el panel recursos, seleccionar el grupo de recursos y luego seleccionar el centro de IoT de la lista de recursos.

1. En **Mensajes**, seleccione **Enrutamiento de mensajes**. En el panel **Enrutamiento de mensajes**, seleccione la pestaña **Puntos de conexión personalizados** y luego seleccione **+ Agregar**. En la lista desplegable, seleccione **Cola de Service Bus**.

   ![Captura de pantalla que resalta la opción de cola de Service Bus.](media/iot-hub-monitoring-notifications-with-azure-logic-apps/select-iot-hub-custom-endpoint.png)

1. En el panel **Agregar un punto de conexión de Service Bus**, escriba la siguiente información:

   **Nombre del punto de conexión**: Nombre del punto de conexión.

   **Espacio de nombres de Service Bus**: seleccione el espacio de nombres que creó.

   **Cola de Service Bus**: seleccione la cola que creó.

   ![Agregar un punto de conexión a IoT Hub en Azure Portal](media/iot-hub-monitoring-notifications-with-azure-logic-apps/3-add-iot-hub-endpoint-azure-portal.png)

1. Seleccione **Crear**. Cuando el punto de conexión se haya creado correctamente, continúe con el paso siguiente.

### <a name="add-a-routing-rule"></a>Agregar una regla de enrutamiento

1. En el panel **Enrutamiento de mensajes**, seleccione la pestaña **Rutas** y seleccione **+ Agregar**.

1. En el panel **Agregar una ruta**, escriba la siguiente información:

   **Name**: nombre de la regla de enrutamiento.

   **Punto de conexión**: seleccione el punto de conexión que creó.

   **Origen de datos**: seleccione **Mensajes de telemetría del dispositivo**.

   **Consulta de enrutamiento**: Escriba `temperatureAlert = "true"`.

   ![Agregar una regla de enrutamiento en Azure Portal](media/iot-hub-monitoring-notifications-with-azure-logic-apps/4-add-routing-rule-azure-portal.png)

1. Seleccione **Guardar**. Puede cerrar el panel **Enrutamiento de mensajes**.

## <a name="create-and-configure-a-logic-app"></a>Crear y configurar una aplicación lógica

En la sección anterior, configuró el centro de IoT para enrutar los mensajes que contienen una alerta de temperatura a la cola de Service Bus. Ahora, configurará una aplicación lógica para supervisar la cola de Service Bus y enviar una notificación por correo electrónico cada vez que se agregue un mensaje a la cola.

### <a name="create-a-logic-app"></a>Creación de una aplicación lógica

1. Seleccione **Crear un recurso** > **Integración** > **Aplicación lógica**.

1. Escriba la siguiente información:

   **Name**: nombre de la aplicación lógica.

   **Grupo de recursos**: use el mismo grupo de recursos que usa el centro de IoT.

   **Ubicación**: use la misma ubicación que emplea el centro de IoT.

   ![Crear una aplicación lógica en Azure Portal](media/iot-hub-monitoring-notifications-with-azure-logic-apps/create-a-logic-app.png)

1. Seleccione **Crear**.

### <a name="configure-the-logic-app-trigger"></a>Configurar el desencadenador de aplicación lógica

1. Abra la aplicación lógica. La manera más fácil llegar a la aplicación lógica es seleccionar **Grupos de recursos** en el panel recursos, seleccionar el grupo de recursos y luego seleccionar la aplicación lógica de la lista de recursos. Cuando se selecciona la aplicación lógica, se abre el Diseñador de aplicaciones lógicas.

1. En el Diseñador de aplicaciones lógicas, desplácese hacia abajo hasta **Plantillas** y seleccione **Aplicación lógica en blanco**.

   ![Comenzar con una aplicación lógica en blanco en Azure Portal](media/iot-hub-monitoring-notifications-with-azure-logic-apps/5-start-with-blank-logic-app-azure-portal.png)

1. Seleccione la pestaña **Todo** y luego seleccione **Service Bus**.

   ![Seleccionar Service Bus para empezar a crear la aplicación lógica en Azure Portal](media/iot-hub-monitoring-notifications-with-azure-logic-apps/6-select-service-bus-when-creating-blank-logic-app-azure-portal.png)

1. En **Desencadenadores**, seleccione **Cuando llegan uno o más mensajes a una cola (autocompletar)** .

   ![Seleccionar el desencadenador para la aplicación lógica en Azure Portal](media/iot-hub-monitoring-notifications-with-azure-logic-apps/select-service-bus-trigger.png)

1. Cree una conexión de Service Bus.
   1. Escriba un nombre de conexión y seleccione el espacio de nombres de Service Bus de la lista. Se abre la pantalla siguiente.

      ![Captura de pantalla que resalta la opción Cuando llegan uno o más mensajes a una cola (autocompletar).](media/iot-hub-monitoring-notifications-with-azure-logic-apps/create-service-bus-connection-1.png)

   1. Seleccione la directiva de Service Bus (RootManageSharedAccessKey). Luego seleccione **Crear**.

      ![Crear una conexión de Service Bus para la aplicación lógica en Azure Portal](media/iot-hub-monitoring-notifications-with-azure-logic-apps/7-create-service-bus-connection-in-logic-app-azure-portal.png)

   1. En la pantalla final, para **Nombre de cola**, seleccione la cola que creó de la lista desplegable. Escriba `175` para **Recuento máximo de mensajes**.

      ![Especificar el recuento máximo de mensajes de la conexión de Service Bus en la aplicación lógica](media/iot-hub-monitoring-notifications-with-azure-logic-apps/8-specify-maximum-message-count-for-service-bus-connection-logic-app-azure-portal.png)

   1. Seleccione **Guardar** en el menú de la parte superior del Diseñador de aplicaciones lógicas para guardar los cambios.

### <a name="configure-the-logic-app-action"></a>Configurar la acción de la aplicación lógica

1. Cree una conexión de servicio SMTP.

   1. Seleccione **Nuevo paso**. En **Elegir una acción**, seleccione la pestaña **Todo**.

   1. Escriba `smtp` en el cuadro de búsqueda, seleccione el servicio **SMTP** en el resultado de búsqueda y luego seleccione **Enviar correo electrónico**.

      ![Crear una conexión SMTP en la aplicación lógica en Azure Portal](media/iot-hub-monitoring-notifications-with-azure-logic-apps/9-create-smtp-connection-logic-app-azure-portal.png)

   1. Escriba la información de SMTP para el buzón y seleccione **Crear**.

      ![Escribir la información de la conexión SMTP en la aplicación lógica en Azure Portal](media/iot-hub-monitoring-notifications-with-azure-logic-apps/10-enter-smtp-connection-info-logic-app-azure-portal.png)

      Obtenga la información SMTP de [Hotmail/Outlook.com](https://support.office.com/article/Add-your-Outlook-com-account-to-another-mail-app-73f3b178-0009-41ae-aab1-87b80fa94970), [Gmail](https://support.google.com/a/answer/176600?hl=en) y [Yahoo Mail](https://help.yahoo.com/kb/SLN4075.html).

      > [!NOTE]
      > Es posible que sea necesario deshabilitar TLS/SSL para establecer la conexión. Si es así y desea volver a habilitar TLS una vez establecida la conexión, consulte el paso opcional al final de esta sección.

   1. En el elemento desplegable **Agregar nuevo parámetro** del paso **Enviar correo electrónico**, seleccione **De**, **Para**, **Asunto** y **Cuerpo**. Haga clic o pulse en cualquier parte en la pantalla para cerrar el cuadro de selección.

      ![Elegir los campos de correo electrónico de conexión SMTP](media/iot-hub-monitoring-notifications-with-azure-logic-apps/smtp-connection-choose-fields.png)

   1. Escriba la dirección de correo electrónico en **De** y **Para** y `High temperature detected` en **Asunto** y **Cuerpo**. Si se abre el cuadro de diálogo **Agregue contenido dinámico de las aplicaciones y conectores que se usan en este flujo**, seleccione **Ocultar** para cerrarlo. No use contenido dinámico en este tutorial.

      ![Rellenar los campos de correo electrónico de conexión SMTP](media/iot-hub-monitoring-notifications-with-azure-logic-apps/fill-in-smtp-connection-fields.png)

   1. Seleccione **Guardar** para guardar la conexión SMTP.

1. (Opcional) Si tuviera que deshabilitar TLS para establecer una conexión con el proveedor de correo electrónico y desea volver a habilitarla, siga estos pasos:

   1. En el panel **Aplicación lógica**, en **Herramientas de desarrollo**, seleccione **Conexiones de API**.

   1. En la lista de conexiones de API, seleccione la conexión SMTP.

   1. En el panel **smtp API Connection** (Conexión de API SMTP), en **General**, seleccione **Editar la conexión de API**.

   1. En el panel **Editar la conexión de API**, seleccione **¿Quiere habilitar SSL?** , vuelva a escribir la contraseña de su cuenta de correo electrónico y seleccione **Guardar**.

      ![Editar una conexión de API de SMTP en la aplicación lógica en Azure Portal](media/iot-hub-monitoring-notifications-with-azure-logic-apps/re-enable-smtp-connection-ssl.png)

La aplicación lógica ahora está lista para procesar las alertas de temperatura de la cola de Service Bus y enviar notificaciones a su cuenta de correo electrónico.

## <a name="test-the-logic-app"></a>Probar la aplicación lógica

1. Inicie la aplicación cliente en el dispositivo.

1. Si usa un dispositivo físico, acerque cuidadosamente una fuente de calor al sensor de calor hasta que la temperatura supere los 30 grados C. Si usa el simulador en línea, el código de cliente generará aleatoriamente los mensajes de telemetría que superan los 30 ºC.

1. Debería empezar a recibir notificaciones por correo electrónico enviadas por la aplicación lógica.

   > [!NOTE]
   > Es posible que el proveedor de servicios de correo electrónico necesite comprobar la identidad del remitente para asegurarse de que es usted quien envía el mensaje.

## <a name="next-steps"></a>Pasos siguientes

Ha creado correctamente una aplicación lógica que conecta IoT Hub y el buzón de correo para la supervisión de temperatura y las notificaciones.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]