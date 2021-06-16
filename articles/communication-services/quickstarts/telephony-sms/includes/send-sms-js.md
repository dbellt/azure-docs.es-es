---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: bertong
manager: ankita
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/11/2021
ms.topic: include
ms.custom: include file
ms.author: bertong
ms.openlocfilehash: b618a2d33b812d964d48618b42421bdce109f28d
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111430228"
---
Introducción a Azure Communication Services mediante el SDK de SMS de Communication Services para JavaScript para enviar mensajes SMS.

Este inicio rápido supone un pequeño costo en su cuenta de Azure.

> [!NOTE]
> Busque el código finalizado de este inicio rápido en [GitHub](https://github.com/Azure-Samples/communication-services-javascript-quickstarts/tree/main/send-sms)

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Versiones de [Node.js](https://nodejs.org/), Active LTS y Maintenance LTS (se recomiendan 8.11.1 y 10.14.1).
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../../create-communication-resource.md).
- Un número de teléfono habilitado para SMS. [Obtención de un número de teléfono](../get-phone-number.md).

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

- En una ventana de terminal o de comandos, ejecute `node --version` para comprobar que Node.js está instalado.
- Para ver los números de teléfono asociados a su recurso de Communication Services, inicie sesión en [Azure Portal](https://portal.azure.com/), busque el recurso de Communication Services y abra la pestaña **números de teléfono** en el panel de navegación izquierdo.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-nodejs-application"></a>Creación de una nueva aplicación Node.js

En primer lugar, abra la ventana de comandos o de terminal, cree un nuevo directorio para la aplicación y navegue hasta este.

```console
mkdir sms-quickstart && cd sms-quickstart
```

Ejecute `npm init -y` para crear un archivo **package.json** con la configuración predeterminada.

```console
npm init -y
```

Use un editor de texto para crear un archivo denominado **send-sms.js** en el directorio raíz del proyecto. En las secciones siguientes agregará todo el código fuente de esta guía de inicio rápido a este archivo.

### <a name="install-the-package"></a>Instalar el paquete

Use el comando `npm install` para instalar el SDK de SMS de Azure Communication Services para JavaScript.

```console
npm install @azure/communication-sms --save
```

La opción `--save` muestra la biblioteca como dependencia en el archivo **package.json**.

## <a name="object-model"></a>Modelo de objetos

Las clases e interfaces siguientes controlan algunas de las características principales del SDK de SMS de Azure Communication Services para Node.js.

| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| SmsClient | Esta clase es necesaria para la funcionalidad de los SMS. Cree una instancia de esta clase con la información de suscripción y úsela para enviar mensajes de texto. |
| SmsSendRequest | Esta interfaz es el modelo para la creación de la solicitud de SMS (por ejemplo, configurar los números de teléfono de origen y destino y el contenido del SMS). |
| SmsSendOptions | Esta interfaz proporciona opciones para configurar los informes de entrega. Si `enableDeliveryReport` se establece en `true`, se emitirá un evento cuando la entrega se realice correctamente. |
| SmsSendResult               | Esta clase contiene el resultado del servicio SMS.                                          |

## <a name="authenticate-the-client"></a>Autenticar el cliente

Importe la clase **SmsClient** desde el SDK y cree una instancia de ella con la cadena de conexión. El código siguiente recupera la cadena de conexión para el recurso de una variable de entorno denominada `COMMUNICATION_SERVICES_CONNECTION_STRING`. Aprenda a [administrar la cadena de conexión del recurso](../../create-communication-resource.md#store-your-connection-string).

Cree y abra un archivo llamado **send-sms.js** y agregue el código siguiente:

```javascript
const { SmsClient } = require('@azure/communication-sms');

// This code demonstrates how to fetch your connection string
// from an environment variable.
const connectionString = process.env['COMMUNICATION_SERVICES_CONNECTION_STRING'];

// Instantiate the SMS client
const smsClient = new SmsClient(connectionString);
```

## <a name="send-a-1n-sms-message"></a>Envío de un SMS de un remitente a varios destinatarios

Para enviar un SMS a una lista de destinatarios, llame a la función `send` desde SmsClient con una lista de números de teléfono de destinatarios (si desea enviar un mensaje a un solo destinatario, incluya solo un número en la lista). Agregue este código al final de **send-sms.js**:

```javascript
async function main() {
  const sendResults = await smsClient.send({
    from: "<from-phone-number>",
    to: ["<to-phone-number-1>", "<to-phone-number-2>"],
    message: "Hello World 👋🏻 via SMS"
  });

  // individual messages can encounter errors during sending
  // use the "successful" property to verify
  for (const sendResult of sendResults) {
    if (sendResult.successful) {
      console.log("Success: ", sendResult);
    } else {
      console.error("Something went wrong when trying to send this message: ", sendResult);
    }
  }
}

main();
```
Debe reemplazar `<from-phone-number>` por un número de teléfono habilitado para SMS asociado al recurso de Communication Services y `<to-phone-number-1>` y `<to-phone-number-2>` por los números de teléfono a los que quiere enviar un mensaje.

> [!WARNING]
> Tenga en cuenta que los números de teléfono se deben proporcionar en formato estándar internacional E.164. (por ejemplo, +14255550123).

## <a name="send-a-1n-sms-message-with-options"></a>Envío de un SMS de un remitente a varios destinatarios con opciones

También puede incluir un objeto de opciones para especificar si el informe de entrega debe estar habilitado y para establecer etiquetas personalizadas.

```javascript

async function main() {
  const sendResults = await smsClient.send({
    from: "<from-phone-number>",
    to: ["<to-phone-number-1>", "<to-phone-number-2>"],
    message: "Weekly Promotion!"
  }, {
    //Optional parameters
    enableDeliveryReport: true,
    tag: "marketing"
  });

  // individual messages can encounter errors during sending
  // use the "successful" property to verify
  for (const sendResult of sendResults) {
    if (sendResult.successful) {
      console.log("Success: ", sendResult);
    } else {
      console.error("Something went wrong when trying to send this message: ", sendResult);
    }
  }
}

main();
```

Debe reemplazar `<from-phone-number>` por un número de teléfono habilitado para SMS asociado al recurso de Communication Services y `<to-phone-number-1>` y `<to-phone-number-2>` por los números de teléfono a los que quiere enviar un mensaje.

> [!WARNING]
> Tenga en cuenta que los números de teléfono se deben proporcionar en formato estándar internacional E.164. (por ejemplo, +14255550123).

El parámetro `enableDeliveryReport` es un parámetro opcional que puede usar para configurar los informes de entrega. Resulta útil para los escenarios en los que quiere emitir eventos cuando se entregan mensajes SMS. Consulte el inicio rápido [Control de eventos SMS](../handle-sms-events.md) a fin de configurar los informes de entrega para los mensajes SMS.
`tag` es un parámetro opcional que puede usar para aplicar una etiqueta al informe de entrega.

## <a name="run-the-code"></a>Ejecución del código

Use el comando `node` para ejecutar el código que agregó al archivo **send-sms.js**.

```console

node ./send-sms.js

```
