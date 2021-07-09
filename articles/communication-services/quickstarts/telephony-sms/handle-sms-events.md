---
title: 'Inicio rápido: Control de eventos SMS para informes de entrega y mensajes entrantes'
titleSuffix: An Azure Communication Services quickstart
description: Obtenga información acerca de cómo controlar eventos SMS mediante Azure Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 06/30/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 04b0ee4feb74553fc3b24910da338ba02864ec28
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113109169"
---
# <a name="quickstart-handle-sms-events-for-delivery-reports-and-inbound-messages"></a>Inicio rápido: Control de eventos SMS para informes de entrega y mensajes entrantes

[!INCLUDE [Regional Availability Notice](../../includes/regional-availability-include.md)]

Para empezar con Azure Communication Services, use Azure Event Grid para administrar eventos SMS de Communication Services.

## <a name="about-azure-event-grid"></a>Acerca de Azure Event Grid

[Azure Event Grid](../../../event-grid/overview.md) es un servicio de eventos basado en la nube. En este artículo, aprenderá a suscribirse a [eventos de servicios de comunicación](../../../event-grid/event-schema-communication-services.md) y a desencadenar un evento para ver el resultado. Por lo general, se envían eventos a un punto de conexión que procesa los datos del evento y realiza acciones. En este artículo, los eventos se envían a una aplicación web que recopila y muestra los mensajes.

## <a name="prerequisites"></a>Requisitos previos
- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un recurso de Azure Communication Services. Puede encontrar más información en la guía de inicio rápido [Creación de un recurso de Azure Communication Services](../create-communication-resource.md).
- Un número de teléfono habilitado para SMS. [Obtener un número de teléfono](./get-phone-number.md).

## <a name="setting-up"></a>Instalación

### <a name="enable-event-grid-resource-provider"></a>Habilitación del proveedor de recursos de Event Grid

Si no ha usado anteriormente Event Grid en su suscripción de Azure, puede que tenga que registrar el proveedor de recursos de Event Grid. Para ello, realice los pasos siguientes:

En Azure Portal:

1. Seleccione **Suscripciones** en el menú de la izquierda.
2. Seleccione la suscripción que está usando para Event Grid.
3. En el menú de la izquierda, en **Configuración**, seleccione **Proveedores de recursos**.
4. Busque **Microsoft.EventGrid**.
5. Si no está registrado, seleccione **Registrar**.

Puede tardar unos instantes en finalizarse el registro. Seleccione **Actualizar**  para actualizar el estado. Cuando el **estado** es **Registrado**, ya está listo para continuar.

### <a name="event-grid-viewer-deployment"></a>Implementación del visor de Event Grid

En esta guía de inicio rápido, usaremos el [ejemplo de visor de Azure Event Grid](/samples/azure-samples/azure-event-grid-viewer/azure-event-grid-viewer/) para ver eventos prácticamente en tiempo real. Esto proporcionará al usuario la experiencia de una fuente en tiempo real. Además, la carga útil de cada evento debe estar disponible también para su inspección.

## <a name="subscribe-to-the-sms-events-using-web-hooks"></a>Suscripción a eventos SMS mediante webhooks

En el portal, navegue hasta el recurso de Azure Communication Services que ha creado. En el recurso de Communication Services, seleccione **Eventos** en el menú de la izquierda de la página **Communication Services**.

:::image type="content" source="./media/handle-sms-events/select-events.png" alt-text="Captura de pantalla que muestra la selección del botón de suscripción a evento en la página de eventos de un recurso.":::

Presione **Add Event Subscription** (Agregar suscripción a evento) para acceder al asistente para creación.

En la página **Crear suscripción de eventos**, escriba un **Nombre** para la suscripción al evento.

Se puede suscribir a eventos específicos para indicar a Event Grid los eventos SMS de los que quiere realizar un seguimiento y el lugar al que deben enviarse. Seleccione los eventos a los que le gustaría suscribirse en el menú desplegable. En el caso de SMS, tendrá la opción de elegir `SMS Received` y `SMS Delivery Report Received`.

Si se le pide que proporcione un **nombre del tema del sistema**, puede proporcionar una cadena única. Este campo no afecta en modo alguno a la experiencia y se usa para telemetría interna.

Consulte la lista completa de [eventos admitidos en Azure Communication Services](../../../event-grid/event-schema-communication-services.md).

:::image type="content" source="./media/handle-sms-events/select-events-create-eventsub.png" alt-text="Captura de pantalla que muestra los tipos de eventos SMS Received (SMS recibidos) y SMS Delivery Report Received (Informe de entrega de SMS recibido) seleccionados.":::

En **Webhook**, seleccione **Tipo de punto de conexión**.

:::image type="content" source="./media/handle-sms-events/select-events-create-linkwebhook.png" alt-text="Captura de pantalla que muestra el campo Tipo de punto de conexión establecido en Webhook.":::

En **Punto de conexión**, haga clic en **Seleccione un punto de conexión** y escriba la dirección URL de la aplicación web.

En este caso, se usará la dirección URL del [ejemplo de visor de Azure Event Grid](/samples/azure-samples/azure-event-grid-viewer/azure-event-grid-viewer/) configurado anteriormente en la guía de inicio rápido. La dirección URL del ejemplo tendrá el formato: `https://{{site-name}}.azurewebsites.net/api/updates`.

Seleccione **Confirmar selección**.

:::image type="content" source="./media/handle-sms-events/select-events-create-selectwebhook-epadd.png" alt-text="Captura de pantalla que muestra la confirmación de un punto de conexión de webhook.":::

## <a name="viewing-sms-events"></a>Visualización de eventos SMS

### <a name="triggering-sms-events"></a>Desencadenamiento de eventos SMS

Para ver los desencadenadores de eventos, se deben generar eventos en primer lugar.

- Los eventos `SMS Received` se generan cuando el número de teléfono de Communication Services recibe un mensaje de texto. Para desencadenar un evento, basta con enviar un mensaje desde el teléfono al número de teléfono asociado al recurso de Communication Services.
- Los eventos `SMS Delivery Report Received` se generan cuando se envía un SMS a un usuario mediante un número de teléfono de Communication Services. Para desencadenar un evento, es necesario que habilite `Delivery Report` en las opciones de [envío de SMS](../telephony-sms/send.md). Intente enviar un mensaje a su teléfono con `Delivery Report`. La realización de esta acción supone un pequeño costo de unos céntimos en su cuenta de Azure.

Consulte la lista completa de [eventos admitidos en Azure Communication Services](../../../event-grid/event-schema-communication-services.md).

### <a name="receiving-sms-events"></a>Recepción de eventos SMS

Cuando complete cualquiera de las acciones anteriores, observará que los eventos `SMS Received` y `SMS Delivery Report Received` se han enviado al punto de conexión. Estos eventos se mostrarán en el [ejemplo de visor de Azure Event Grid](/samples/azure-samples/azure-event-grid-viewer/azure-event-grid-viewer/) que configuramos al principio. Puede presionar el icono de ojo situado junto al evento para ver toda la carga útil. Este será el aspecto de los eventos:

:::image type="content" source="./media/handle-sms-events/sms-received.png" alt-text="Captura de pantalla que muestra el esquema de Event Grid para un evento recibido de SMS.":::

:::image type="content" source="./media/handle-sms-events/sms-delivery-report-received.png" alt-text="Captura de pantalla que muestra el esquema de Event Grid para un evento de informe de entrega de SMS.":::

Más información sobre los [esquemas de eventos y otros conceptos de eventos](../../../event-grid/event-schema-communication-services.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y quitar una suscripción a Communication Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él. Obtenga más información sobre la [limpieza de recursos](../create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a utilizar eventos SMS. Puede recibir mensajes SMS mediante la creación de una suscripción de Event Grid.

> [!div class="nextstepaction"]
> [Envío de SMS](../telephony-sms/send.md)

Puede que también le interese:

 - [Más información sobre los conceptos de control de eventos](../../../event-grid/event-schema-communication-services.md)
 - [Información sobre Event Grid](../../../event-grid/overview.md)