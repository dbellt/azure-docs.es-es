---
title: Registros de Communication Services
titleSuffix: An Azure Communication Services concept document
description: Más información sobre el registro en Azure Communication Services
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 06/30/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: b8f263a1fefd3a75333f540f1f6e8e5ec1c6870b
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113112264"
---
# <a name="communication-services-logs"></a>Registros de Communication Services

Azure Communication Services ofrece funcionalidades de registro que sirven para supervisar y depurar la solución de Communication Services. Estas funcionalidades pueden configurarse mediante Azure Portal.

## <a name="enable-diagnostic-logs-in-your-resource"></a>Habilitación de los registros de diagnóstico en el recurso

El registro está desactivado de forma predeterminada cuando se crea un recurso. Para habilitarlo, vaya a la hoja **Configuración de diagnóstico** del menú de recursos en la sección **Supervisión**. Haga clic en **Add diagnostic setting** (Agregar configuración de diagnóstico).

Después, establezca el destino de archivo que desee. Actualmente se admiten cuentas de almacenamiento y Log Analytics como destinos de archivo. Después de seleccionar los tipos de registros que desea capturar, guarde la configuración de diagnóstico.
 
La nueva configuración surte efecto en unos diez minutos. Los registros comenzarán a aparecer en el destino de archivo configurado en el panel registros del recurso de Communication Services.

:::image type="content" source="./media/diagnostic-settings.png" alt-text="Opciones de configuración de diagnóstico de ACS":::.

Para más información sobre la configuración de diagnóstico, consulte la información general sobre los [registros de los recursos de Azure](../../azure-monitor/essentials/platform-logs-overview.md).

## <a name="resource-log-categories"></a>Categorías del registro de recursos

Communication Services ofrece tres tipos de registros que se pueden habilitar:

* **Usage logs** (Registros de uso): proporciona datos de uso asociados a cada oferta de servicio facturada.
* **Chat operational logs** (Registros operativos de chat): proporciona información básica relacionada con el servicio de chat.
* **SMS operational logs** (Registros operativos de SMS): proporciona información básica relacionada con el servicio de SMS.
* **Authentication operational logs** (Registros operativos de autenticación): proporciona información básica relacionada con el servicio de autenticación.

### <a name="usage-logs-schema"></a>Esquema de los registros de uso

| Propiedad | Descripción |
| -------- | ---------------|
| Timestamp | Marca de tiempo (UTC) de cuando se generó el registro. |
| Nombre de la operación | Operación asociada al registro. |
| Versión de la operación | `api-version` asociada con la operación, si operationName se ha realizado mediante una API. Si no hay ninguna API que se corresponde con esta operación, la versión representa la versión de esa operación en caso de que las propiedades asociadas con la operación cambien en el futuro. |
| Category | Categoría de registro del evento. La categoría es la granularidad con la que se pueden habilitar o deshabilitar los registros en un recurso determinado. Las propiedades que aparecen en el blob de propiedades de un evento son las mismas dentro de una categoría de registro y un tipo de recurso concretos. |
| Id. de correlación | Identificador de los eventos correlacionados. Se puede utilizar para identificar eventos correlacionados en varias tablas. |
| Propiedades | Otros datos aplicables a distintos modos de Communication Services. |
| Record ID | Identificador único de un registro de uso determinado. |
| Usage Type (Tipo de uso) | Modo de uso. (Por ejemplo, chat, RTC, NAT, etc.). |
| Unit (Tipo) | Tipo de unidad que se emplea para el uso en un modo determinado. (Por ejemplo, minutos, megabytes, mensajes, etc.). |
| Cantidad | Número de unidades usadas o consumidas en este registro. |

### <a name="chat-operational-logs"></a>Registros operativos de chat

| Propiedad | Descripción |
| -------- | ---------------|
| TimeGenerated | Marca de tiempo (UTC) de cuando se generó el registro. |
| OperationName | Operación asociada al registro. |
| CorrelationID | Identificador de los eventos correlacionados. Se puede utilizar para identificar eventos correlacionados en varias tablas. |
| OperationVersion | api-version asociada con la operación, si operationName se ha realizado mediante una API. Si no hay ninguna API que se corresponde con esta operación, la versión representa la versión de esa operación en caso de que las propiedades asociadas con la operación cambien en el futuro. |
| Category | Categoría de registro del evento. La categoría es la granularidad con la que se pueden habilitar o deshabilitar los registros en un recurso determinado. Las propiedades que aparecen en el blob de propiedades de un evento son las mismas dentro de una categoría de registro y un tipo de recurso concretos. |
| ResultType | Estado de la operación. |
| ResultSignature | Subestado de la operación. Si esta operación se corresponde con una llamada API de REST, este campo es el código de estado HTTP de la llamada REST correspondiente. |
| ResultDescription | Descripción de texto estático de esta operación. |
| DurationMs | Duración de la operación en milisegundos. |
| CallerIpAddress | Dirección IP del autor de la llamada, si la operación se corresponde con una llamada API que podría proceder de una entidad con una dirección IP disponible públicamente. |
| Nivel | Nivel de gravedad del evento. |
| URI | URI de la solicitud. |
| UserId | Identificador de usuario del remitente de la solicitud. |
| ChatThreadId | Identificador de la conversación de chat asociado a la solicitud. |
| ChatMessageId | Identificador de mensaje de chat asociado a la solicitud. |
| SdkType | Tipo de SDK que se usa en la solicitud. |
| PlatformType | Tipo de plataforma utilizado en la solicitud. |

### <a name="sms-operational-logs"></a>Registros operativos de SMS

| Propiedad | Descripción |
| -------- | ---------------|
| TimeGenerated | Marca de tiempo (UTC) de cuando se generó el registro. |
| OperationName | Operación asociada al registro. |
| CorrelationID | Identificador de los eventos correlacionados. Se puede utilizar para identificar eventos correlacionados en varias tablas. |
| OperationVersion | api-version asociada con la operación, si operationName se ha realizado mediante una API. Si no hay ninguna API que se corresponde con esta operación, la versión representa la versión de esa operación en caso de que las propiedades asociadas con la operación cambien en el futuro. |
| Category | Categoría de registro del evento. La categoría es la granularidad con la que se pueden habilitar o deshabilitar los registros en un recurso determinado. Las propiedades que aparecen en el blob de propiedades de un evento son las mismas dentro de una categoría de registro y un tipo de recurso concretos. |
| ResultType | Estado de la operación. |
| ResultSignature | Subestado de la operación. Si esta operación se corresponde con una llamada API de REST, este campo es el código de estado HTTP de la llamada REST correspondiente. |
| ResultDescription | Descripción de texto estático de esta operación. |
| DurationMs | Duración de la operación en milisegundos. |
| CallerIpAddress | Dirección IP del autor de la llamada, si la operación se corresponde con una llamada API que podría proceder de una entidad con una dirección IP disponible públicamente. |
| Nivel | Nivel de gravedad del evento. |
| URI | URI de la solicitud. |
| OutgoingMessageLength | Número de caracteres del mensaje saliente. |
| IncomingMessageLength | Número de caracteres del mensaje entrante. |
| DeliveryAttempts | Número de intentos realizados para entregar el mensaje. |
| PhoneNumber | Número de teléfono desde el que se envía el mensaje SMS. |
| SdkType | Tipo de SDK que se usa en la solicitud. |
| PlatformType | Tipo de plataforma utilizado en la solicitud. |
| Método | Método usado para realizar la solicitud. |

### <a name="authentication-operational-logs"></a>Registros operativos de autenticación

| Propiedad. | Descripción |
| -------- | ---------------|
| TimeGenerated | Marca de tiempo (UTC) de cuando se generó el registro. |
| OperationName | Operación asociada al registro. |
| CorrelationID | Identificador de los eventos correlacionados. Se puede utilizar para identificar eventos correlacionados en varias tablas. |
| OperationVersion | `api-version` asociada con la operación, si `operationName` se realizó mediante una API. Si no hay ninguna API que se corresponde con esta operación, la versión representa la versión de esa operación en caso de que las propiedades asociadas con la operación cambien en el futuro. |
| Category | Categoría de registro del evento. La categoría es la granularidad con la que se pueden habilitar o deshabilitar los registros en un recurso determinado. Las propiedades que aparecen en el blob de propiedades de un evento son las mismas dentro de una categoría de registro y un tipo de recurso concretos. |
| ResultType | Estado de la operación. |
| ResultSignature | El subestado de la operación. Si esta operación se corresponde con una llamada API de REST, este campo es el código de estado HTTP de la llamada REST correspondiente. |
| DurationMs | Duración de la operación en milisegundos. |
| CallerIpAddress | Dirección IP del autor de la llamada, si la operación se corresponde con una llamada API que podría proceder de una entidad con una dirección IP disponible públicamente. |
| Nivel | Nivel de gravedad del evento. |
| URI | URI de la solicitud. |
| SdkType | Tipo de SDK que se usa en la solicitud. |
| PlatformType | Tipo de plataforma utilizado en la solicitud. |
| Identidad | La identidad de Communication Services relacionada con la operación. |
| Ámbitos | Los ámbitos de Communication Services presentes en el token de acceso. |
