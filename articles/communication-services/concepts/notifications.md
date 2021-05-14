---
title: Notificaciones en Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Envíe notificaciones a los usuarios de aplicaciones basadas en Azure Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 1a2d4059f701a45d982f2045dea5e8dfe4e12a20
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108286355"
---
# <a name="communication-services-notifications"></a>Notificaciones de Communication Services

Los SDK de llamadas y chat de Azure Communication Services crean un canal de mensajería en tiempo real que permite insertar mensajes de señalización en los clientes conectados de una manera eficaz y confiable. De este modo, puede incluir una funcionalidad de comunicación enriquecida y en tiempo real en sus aplicaciones sin necesidad de implementar una lógica complicada de sondeo HTTP. Sin embargo, en las aplicaciones móviles, este canal de señalización solo permanece conectado si la aplicación está activa en primer plano. Si quiere que los usuarios reciban llamadas entrantes o mensajes de chat mientras la aplicación está en segundo plano, debe usar las notificaciones push.

Las notificaciones push permiten enviar información de la aplicación a los dispositivos móviles de los usuarios. Puede usar las notificaciones push para mostrar un cuadro de diálogo, reproducir un sonido o mostrar la interfaz de usuario de llamadas entrantes. Azure Communication Services ofrece integraciones con [Azure Event Grid](../../event-grid/overview.md) y [Azure Notification Hubs](../../notification-hubs/notification-hubs-push-notification-overview.md) que permiten agregar notificaciones push a las aplicaciones.

## <a name="trigger-push-notifications-via-azure-event-grid"></a>Desencadenamiento de notificaciones a través de Azure Event Grid

Azure Communication Services se integra con [Azure Event Grid](https://azure.microsoft.com/services/event-grid/) para ofrecer notificaciones de eventos en tiempo real de manera confiable, escalable y segura. Puede aprovechar esta integración para crear un servicio de notificaciones que entregue notificaciones push móviles a los usuarios al crear una suscripción a Event Grid que desencadene una [función de Azure Functions](../../azure-functions/functions-overview.md) o un webhook.

:::image type="content" source="./media/notifications/acs-events-int.png" alt-text="Diagrama que muestra cómo se integra Communication Services con Event Grid.":::

Obtenga más información sobre el [control de eventos en Azure Communication Services](../../event-grid/event-schema-communication-services.md).

## <a name="deliver-push-notifications-via-azure-notification-hubs"></a>Entrega de notificaciones push mediante Azure Notification Hubs

Puede conectar una instancia de Azure Notification Hubs a su recurso de Communication Services para enviar automáticamente notificaciones push al dispositivo móvil de un usuario cuando recibe una llamada entrante. Estas notificaciones push se deben usar para reactivar la aplicación en segundo plano y mostrar la interfaz de usuario que permite al usuario aceptar o rechazar la llamada.

:::image type="content" source="./media/notifications/acs-anh-int.png" alt-text="Diagrama que muestra cómo se integra Communication Services con Azure Notification Hubs.":::

Communication Services usa Azure Notification Hubs como un servicio de tránsito para comunicarse con los diversos servicios de notificaciones push específicos de cada plataforma mediante [Direct Send](/rest/api/notificationhubs/direct-send) API. De este modo, podrá volver a usar los recursos y valores de configuración de Azure Notification Hubs existentes para ofrecer notificaciones de llamada confiables y de baja latencia a sus aplicaciones.

> [!NOTE]
> Actualmente solo se admite la llamada a las notificaciones push.

### <a name="notification-hub-provisioning"></a>Aprovisionamiento de una instancia de Notification Hubs

Para enviar notificaciones push a dispositivos cliente mediante Notification Hubs, [cree una instancia de Notification Hubs](../../notification-hubs/create-notification-hub-portal.md) dentro de la misma suscripción en la que se encuentra el recurso de Communication Services. Azure Notification Hubs se debe configurar específicamente para el Sistema de notificación de plataforma que se desee usar. Para información sobre cómo obtener notificaciones de inserción en la aplicación cliente desde Notification Hubs, consulte [Introducción a Notification Hubs](../../notification-hubs/notification-hubs-android-push-notification-google-fcm-get-started.md) y seleccione la plataforma cliente de destino en la lista desplegable que aparece cerca de la parte superior de la página.

> [!NOTE]
> Actualmente se admiten las plataformas APNs y FCM.
La plataforma de APNs debe configurarse con el modo de autenticación de tokens. El modo de autenticación de certificados no se admite por ahora.

Una vez configurada la instancia de Notification Hubs, puede asociarla a su recurso de Communication Services; para ello, proporcione una cadena de conexión para el centro mediante el cliente de Azure Resource Manager o Azure Portal. La cadena de conexión debe contener permisos `Send`. Se recomienda crear otra directiva de acceso con permisos `Send` únicamente para el centro. Más información sobre las [directivas de seguridad y acceso de Notification Hubs](../../notification-hubs/notification-hubs-push-notification-security.md)

#### <a name="using-the-azure-resource-manager-client-to-link-your-notification-hub"></a>Uso del cliente de Azure Resource Manager para vincular una instancia de Notification Hubs

Para iniciar sesión en Azure Resource Manager, ejecute el siguiente código e inicie sesión con sus credenciales.

```console
armclient login
```

 Una vez que haya iniciado sesión correctamente, ejecute el siguiente código para aprovisionar el centro de notificaciones:

```console
armclient POST /subscriptions/<sub_id>/resourceGroups/<resource_group>/providers/Microsoft.Communication/CommunicationServices/<resource_id>/linkNotificationHub?api-version=2020-08-20-preview "{'connectionString': '<connection_string>','resourceId': '<resource_id>'}"
```

#### <a name="using-the-azure-portal-to-link-your-notification-hub"></a>Uso de Azure Portal para vincular una instancia de Notification Hubs

1. En el portal, vaya al recurso de Azure Communication Services.

1. En el recurso de Communication Services, seleccione **Notificaciones push** en el menú de la izquierda de la página Communication Services y conecte la instancia de Notification Hubs que aprovisionó anteriormente.

1. Seleccione **Connect notification hub** (Conectar el centro de notificaciones). Verá una lista de centros de notificaciones disponibles para conectarse.
 
1. Seleccione el centro de notificaciones que desee usar para este recurso.
 
   - Si necesita crear un nuevo centro de notificaciones, seleccione **Crear un nuevo Centro de notificaciones** para obtener un nuevo centro aprovisionado para este recurso.

   :::image type="content" source="./media/notifications/acs-anh-portal-int.png" alt-text="Captura de pantalla que muestra la configuración de notificaciones push en Azure Portal.":::

Ahora verá el centro de notificaciones vinculado con el estado conectado.

Si desea usar otro centro para el recurso, seleccione **Desconectar** y, a continuación, repita los pasos para vincular el otro centro de notificaciones.

> [!NOTE]
> Cualquier cambio en la forma en que se vincula el centro se reflejará en el plano de datos (es decir, al enviar una notificación) en un período máximo de 10 minutos. Este mismo comportamiento es aplicable cuando el centro se vincula por primera vez, **si** se enviaron notificaciones antes del cambio.

### <a name="device-registration"></a>Registro de dispositivos

Consulte la [guía de inicio rápido de llamada de voz](../quickstarts/voice-video-calling/getting-started-with-calling.md) para obtener información sobre cómo registrar el identificador del dispositivo con Communication Services.

### <a name="troubleshooting-guide-for-push-notifications"></a>Guía para la solución de problemas en las notificaciones push

Cuando no se ven notificaciones push en un dispositivo, se pueden haber colocado en uno de estos tres lugares:

- Azure Notification Hubs no ha aceptado la notificación de Azure Communication Services
- El Sistema de notificación de plataforma (por ejemplo, Apple Push Notification Service y FCM) no ha aceptado la notificación de Azure Notification Hubs
- El Sistema de notificación de plataforma no ha entregado la notificación al dispositivo.

A continuación se trata el primer lugar en el que se puede haber colocado una notificación (Azure Notification Hubs no ha aceptado las notificaciones de Azure Communication Services). En cuanto a los otros dos lugares, consulte [Diagnóstico de notificaciones eliminadas en Azure Notification Hubs](../../notification-hubs/notification-hubs-push-notification-fixer.md).

Una forma de ver si un recurso de Communication Services envía notificaciones a Azure Notification Hubs es examinar la métrica `incoming messages` de las [métricas de Azure Notification Hubs](../../azure-monitor/essentials/metrics-supported.md#microsoftnotificationhubsnamespacesnotificationhubs) vinculadas.

A continuación se muestran algunos errores de configuración comunes que podrían ser la causa de que Azure Notification Hubs no acepte las notificaciones del recurso de Communication Services.

#### <a name="azure-notification-hub-not-linked-to-the-communication-services-resource"></a>Azure Notification Hubs no está vinculado al recurso de Communication Services

Podría deberse a que no ha vinculado su instancia de Azure Notification Hubs a su recurso de Communication Services. Para ver cómo se vinculan, consulte la [sección de aprovisionamiento de Notification Hubs](#notification-hub-provisioning).

#### <a name="the-linked-azure-notification-hub-isnt-configured"></a>La instancia de Azure Notification Hubs vinculada no está configurada

Tiene que configurar la instancia de Azure Notification Hubs vinculada con las credenciales del Sistema de notificación de plataforma de la plataforma (por ejemplo, iOS o Android) que desee usar. Para más información sobre cómo hacerlo, consulte [Configuración de notificaciones push en un centro de notificaciones](../../notification-hubs/configure-notification-hub-portal-pns-settings.md).

#### <a name="the-linked-azure-notification-hub-doesnt-exist"></a>La instancia de Azure Notification Hubs vinculada no existe

La instancia de Azure Notification Hubs vinculada a su recurso de Communication Services ya no existe. Compruebe que la instancia de Azure Notification Hubs vinculada sigue existiendo.

#### <a name="the-azure-notification-hub-apns-platform-is-configured-with-certificate-authentication-mode"></a>La plataforma de Apple Push Notification Service de Azure Notification Hubs se configura con el modo de autenticación de certificados

Aunque desea usar la plataforma de Apple Push Notification Service con el modo de autenticación de certificados no puede hacerlo, ya que actualmente no se admite. Debe configurar la plataforma de Apple Push Notification Service con el modo de autenticación de tokens tal como se especifica en [configurar notificaciones Push en un centro de notificaciones](../../notification-hubs/configure-notification-hub-portal-pns-settings.md).

#### <a name="the-linked-connection-string-doesnt-have-send-permission"></a>La cadena de conexión vinculada no tiene el permiso `Send`

La cadena de conexión que usó para vincular Notification Hubs a su recurso de Communication Services debe tener el permiso `Send`. Para más información sobre cómo crear una cadena de conexión o ver la cadena de conexión actual desde Azure Notification Hubs, puede consultar [Directivas de seguridad y acceso de Notification Hubs](../../notification-hubs/notification-hubs-push-notification-security.md).

#### <a name="the-linked-connection-string-or-azure-notification-hub-resourceid-arent-valid"></a>La cadena de conexión vinculada o el resourceId de Azure Notification Hubs no son válidos

Asegúrese de configurar el recurso de Communication Services con la cadena de conexión correcta y el resourceId de Communication Services

#### <a name="the-linked-connection-string-is-regenerated"></a>Se vuelve a generar la cadena de conexión vinculada

En caso de que haya vuelto a generar la cadena de conexión de la instancia de Azure Notification Hubs vinculada, tendrá que actualizar la cadena de conexión con la nueva en el recurso de Communication Services, para lo que deberá [volver a vincular Notification Hubs](#notification-hub-provisioning).

## <a name="next-steps"></a>Pasos siguientes

* Para una introducción a Azure Event Grid, consulte [Introducción a Azure Event Grid](../../event-grid/overview.md).
* Para más información sobre los conceptos de Azure Notification Hubs, consulte la [documentación de Azure Notification Hubs](../../notification-hubs/index.yml)