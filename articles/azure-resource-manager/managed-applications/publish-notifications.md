---
title: Aplicaciones administradas con notificaciones
description: Configure aplicaciones administradas con puntos de conexión de webhook para recibir notificaciones relativas a las operaciones de creación, actualización y eliminación, así como a los errores que se producen en las instancias de aplicaciones administradas.
ms.topic: conceptual
ms.author: ilahat
author: ilahat
ms.date: 11/01/2019
ms.openlocfilehash: 229abeb71a580444525a314f648ee09452585336
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111971933"
---
# <a name="azure-managed-applications-with-notifications"></a>Aplicaciones administradas de Azure con notificaciones

Las notificaciones de aplicaciones administradas de Azure permiten a los publicadores automatizar acciones basadas en eventos del ciclo de vida de las instancias de la aplicación administrada. Los publicadores pueden especificar puntos de conexión de webhook de notificación personalizados para recibir notificaciones de eventos sobre las instancias de aplicaciones administradas nuevas y existentes. Los publicadores pueden configurar flujos de trabajo personalizados en el momento del aprovisionamiento, las actualizaciones y la eliminación de la aplicación.

## <a name="getting-started"></a>Introducción
Para empezar a recibir aplicaciones administradas, ponga en marcha un punto de conexión HTTPS público y especifíquelo al publicar la definición de la aplicación de catálogo de servicios o la oferta de Azure Marketplace.

Estos son los pasos recomendados para empezar a usarlas rápidamente:
1. Ponga en marcha un punto de conexión HTTPS público que registre las solicitudes POST entrantes y devuelva `200 OK`.
2. Agregue el punto de conexión a la definición de aplicación de catálogo de servicios o a la oferta de Azure Marketplace, como se explica más adelante en este artículo.
3. Cree una instancia de aplicación administrada que haga referencia a la definición de la aplicación o a la oferta de Azure Marketplace.
4. Confirme que las notificaciones se reciben.
5. Habilite la autorización tal y como se explica en la sección **Autenticación de punto de conexión** de este artículo.
6. Siga las instrucciones de la sección **Esquema de notificación** de este artículo para analizar las solicitudes de notificación e implementar la lógica de negocios basada en la notificación.

## <a name="add-service-catalog-application-definition-notifications"></a>Adición de notificaciones de definición de aplicación del catálogo de servicios
#### <a name="azure-portal"></a>Portal de Azure
Para empezar, vea [Publicación de una aplicación de catálogo de servicios mediante Azure Portal](./publish-portal.md).

![Notificaciones de definición de aplicación del catálogo de servicios en Azure Portal](./media/publish-notifications/service-catalog-notifications.png)

#### <a name="rest-api"></a>API DE REST

> [!NOTE]
> Actualmente, solo se puede proporcionar un punto de conexión en `notificationEndpoints`, en las propiedades de definición de la aplicación.

``` JSON
    {
      "properties": {
        "isEnabled": true,
        "lockLevel": "ReadOnly",
        "displayName": "Sample Application Definition",
        "description": "Notification-enabled application definition.",
        "notificationPolicy": {
            "notificationEndpoints": [
                {
                    "uri": "https://isv.azurewebsites.net:1214?sig=unique_token"
                }
            ]
        },
        "authorizations": [
          {
            "principalId": "d6b7fbd3-4d99-43fe-8a7a-f13aef11dc18",
            "roleDefinitionId": "8e3af657-a8ff-443c-a75c-2fe8c4bcb635"
          },
        ...

```
## <a name="add-azure-marketplace-managed-application-notifications"></a>Adición de notificaciones de aplicaciones administradas de Azure Marketplace
Para más información, vea [Crear una oferta de una aplicación de Azure](../../marketplace/azure-app-offer-setup.md).

![Notificaciones de aplicaciones administradas de Azure Marketplace en Azure Portal](./media/publish-notifications/marketplace-notifications.png)
## <a name="event-triggers"></a>Desencadenadores de eventos
En la siguiente tabla se describen todas las posibles combinaciones de EventType y ProvisioningState y sus desencadenadores:

EventType | ProvisioningState | Desencadenador para la notificación
---|---|---
PUT | Accepted | Se ha creado y proyectado correctamente un grupo de recursos administrados después del evento PUT de la aplicación (antes de que se inicie la implementación en el grupo de recursos administrados).
PUT | Correcto | El aprovisionamiento completo de la aplicación administrada se realizó correctamente después de un PUT.
PUT | Con error | Error al poner el aprovisionamiento de la instancia de aplicación en cualquier momento.
PATCH | Correcto | Después de un evento PATCH correcto de la instancia de la aplicación administrada para actualizar las etiquetas, la directiva de acceso JIT o la identidad administrada.
Delete | Eliminando | En cuanto el usuario inicia una eliminación de una instancia de la aplicación administrada.
Delete | Deleted | Después de la eliminación completa y correcta de la aplicación administrada.
Delete | Con error | Después de cualquier error durante el proceso de desaprovisionamiento que bloquea la eliminación.
## <a name="notification-schema"></a>Esquema de la notificación
Al poner en marcha el punto de conexión de webhook para controlar las notificaciones, necesitará analizar la carga para obtener propiedades importantes y, después, actuar sobre la notificación. Las notificaciones de aplicaciones administradas del catálogo de servicios y de Azure Marketplace proporcionan muchas de esas mismas propiedades, salvo por dos pequeñas diferencias incluidas en la tabla que hay después de los ejemplos.

#### <a name="service-catalog-application-notification-schema"></a>Esquema de notificación de aplicación de catálogo de servicios
Este es un ejemplo de notificación del catálogo de servicios de ejemplo después de un aprovisionamiento correcto de una instancia de la aplicación administrada:
``` HTTP
POST https://{your_endpoint_URI}/resource?{optional_parameter}={optional_parameter_value} HTTP/1.1

{
    "eventType": "PUT",
    "applicationId": "/subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applications/<applicationName>",
    "eventTime": "2019-08-14T19:20:08.1707163Z",
    "provisioningState": "Succeeded",
    "applicationDefinitionId": "/subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applicationDefinitions/<appDefName>"    
}

```

Si se produce un error en el aprovisionamiento, se enviará una notificación con los detalles del error al punto de conexión especificado.

``` HTTP
POST https://{your_endpoint_URI}/resource?{optional_parameter}={optional_parameter_value} HTTP/1.1

{
    "eventType": "PUT",
    "applicationId": "subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applications/<applicationName>",
    "eventTime": "2019-08-14T19:20:08.1707163Z",
    "provisioningState": "Failed",
    "applicationDefinitionId": "/subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applicationDefinitions/<appDefName>",
    "error": {
        "code": "ErrorCode",
        "message": "error message",
        "details": [
            {
                "code": "DetailedErrorCode",
                "message": "error message"
            }
        ]
    }
}

```

#### <a name="azure-marketplace-application-notification-schema"></a>Esquema de notificación de aplicación de Azure Marketplace

Este es un ejemplo de notificación del catálogo de servicios de ejemplo después de un aprovisionamiento correcto de una instancia de la aplicación administrada:
``` HTTP
POST https://{your_endpoint_URI}/resource?{optional_parameter}={optional_parameter_value} HTTP/1.1

{
    "eventType": "PUT",
    "applicationId": "/subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applications/<applicationName>",
    "eventTime": "2019-08-14T19:20:08.1707163Z",
    "provisioningState": "Succeeded",
    "billingDetails": {
        "resourceUsageId":"<resourceUsageId>"
    },
    "plan": {
        "publisher": "publisherId",
        "product": "offer",
        "name": "skuName",
        "version": "1.0.1"
    }
}

```

Si se produce un error en el aprovisionamiento, se enviará una notificación con los detalles del error al punto de conexión especificado.

``` HTTP
POST https://{your_endpoint_URI}/resource?{optional_parameter}={optional_parameter_value} HTTP/1.1

{
    "eventType": "PUT",
    "applicationId": "/subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.Solutions/applications/<applicationName>",
    "eventTime": "2019-08-14T19:20:08.1707163Z",
    "provisioningState": "Failed",
    "billingDetails": {
        "resourceUsageId":"<resourceUsageId>"
    },
    "plan": {
        "publisher": "publisherId",
        "product": "offer",
        "name": "skuName",
        "version": "1.0.1"
    },
    "error": {
        "code": "ErrorCode",
        "message": "error message",
        "details": [
            {
                "code": "DetailedErrorCode",
                "message": "error message"
            }
        ]
    }
}

```

Parámetro | Descripción
---|---
eventType | Tipo de evento que desencadenó la notificación. (Por ejemplo, PUT, PATCH, DELETE).
applicationId | El identificador de recurso completo de la aplicación administrada para la que se desencadenó la notificación.
eventTime | Marca de tiempo del evento que desencadenó la notificación. (Fecha y hora en formato UTC ISO 8601).
provisioningState | El estado de aprovisionamiento de la instancia de la aplicación administrada. (Por ejemplo, Succeeded, Failed, Deleting, Deleted).
error | *Solo se especifica cuando se produce un error en provisioningState*. Contiene el código de error, el mensaje y los detalles del problema que causó el error.
applicationDefinitionId | *Solo se especifica en el caso de las aplicaciones administradas del catálogo de servicios*. Representa el identificador de recurso completo de la definición de la aplicación para la que se aprovisionó la instancia de la aplicación administrada.
plan | *Solo se especifica en el caso de las aplicaciones administradas de Azure Marketplace*. Representa el publicador, la oferta, la SKU y la versión de la instancia de la aplicación administrada.
billingDetails | *Solo se especifica en el caso de las aplicaciones administradas de Azure Marketplace*. El estado de facturación de la instancia de la aplicación administrada. Contiene el identificador resourceUsageId, que sirve para consultar los detalles de uso de Azure Marketplace.

## <a name="endpoint-authentication"></a>Autenticación de punto de conexión
Para proteger el punto de conexión del webhook y garantizar la autenticidad de la notificación:
1. Proporcione un parámetro de consulta en la parte superior del URI del webhook, como este: https\://your-endpoint.com?sig=Guid. Con cada notificación, compruebe que el parámetro de consulta `sig` tiene el valor esperado `Guid`.
2. Emita un evento GET en la instancia de la aplicación administrada con applicationId. Compruebe que el provisioningState coincide con el provisioningState de la notificación para garantizar la coherencia.

## <a name="notification-retries"></a>Reintentos de notificación

El servicio de notificaciones de aplicaciones administradas espera una respuesta `200 OK` del punto de conexión de webhook a la notificación. El servicio de notificación volverá a intentarlo si el punto de conexión de webhook devuelve un código de error HTTP igual o superior a 500, si devuelve un código de error 429 o si el punto de conexión no está disponible temporalmente. Si el punto de conexión de webhook no está disponible en 10 horas, se quitará el mensaje de notificación y se detendrán los reintentos.