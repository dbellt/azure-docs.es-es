---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: tomaschladek
manager: nmurav
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/10/2021
ms.topic: include
ms.custom: include file
ms.author: tchladek
ms.openlocfilehash: 68b5fc7c958f611c43ba3f38bf30ceb63608886a
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/01/2021
ms.locfileid: "106113267"
---
## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Versiones de [Node.js](https://nodejs.org/), Active LTS y Maintenance LTS (se recomiendan 8.11.1 y 10.14.1).
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../create-communication-resource.md).

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-nodejs-application"></a>Creación de una nueva aplicación Node.js

Abra la ventana de comandos o de terminal, cree un nuevo directorio para la aplicación y navegue hasta este.

```console
mkdir access-tokens-quickstart && cd access-tokens-quickstart
```

Ejecute `npm init -y` para crear un archivo **package.json** con la configuración predeterminada.

```console
npm init -y
```

### <a name="install-the-package"></a>Instalar el paquete

Use el comando `npm install` para instalar Identity SDK de Azure Communication Services para JavaScript.

```console

npm install @azure/communication-identity --save

```

La opción `--save` muestra la biblioteca como dependencia en el archivo **package.json**.

## <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Desde el directorio del proyecto:

1. Abra un nuevo archivo de texto en el editor de código.
1. Agregue una llamada `require` para cargar el `CommunicationIdentityClient`.
1. Cree la estructura del programa, incluido un control de excepciones básico.

Use el código siguiente para empezar:

```javascript
const { CommunicationIdentityClient } = require('@azure/communication-identity');

const main = async () => {
  console.log("Azure Communication Services - Access Tokens Quickstart")

  // Quickstart code goes here
};

main().catch((error) => {
  console.log("Encountered an error");
  console.log(error);
})
```

1. Guarde el nuevo archivo como **issue-access-token.js** en el directorio *access-tokens-quickstart*.

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una instancia de un objeto `CommunicationIdentityClient` con su cadena de conexión. El código siguiente recupera la cadena de conexión para el recurso de una variable de entorno denominada `COMMUNICATION_SERVICES_CONNECTION_STRING`. Aprenda a [administrar la cadena de conexión del recurso](../create-communication-resource.md#store-your-connection-string).

Agregue el código siguiente al método `main`:

```javascript
// This code demonstrates how to fetch your connection string
// from an environment variable.
const connectionString = process.env['COMMUNICATION_SERVICES_CONNECTION_STRING'];

// Instantiate the identity client
const identityClient = new CommunicationIdentityClient(connectionString);
```

Como alternativa, puede separar el punto de conexión y la clave de acceso.
```javascript
// This code demonstrates how to fetch your endpoint and access key
// from an environment variable.
const endpoint = process.env["COMMUNICATION_SERVICES_ENDPOINT"];
const accessKey = process.env["COMMUNICATION_SERVICES_ACCESSKEY"];
const tokenCredential = new AzureKeyCredential(accessKey);
// Instantiate the identity client
const identityClient = new CommunicationIdentityClient(endpoint, tokenCredential)
```

Si ha configurado una identidad administrada, consulte [Uso de identidades administradas](../managed-identity.md); también puede autenticarse con la identidad administrada.
```javascript
const endpoint = process.env["COMMUNICATION_SERVICES_ENDPOINT"];
const tokenCredential = new DefaultAzureCredential();
const identityClient = new CommunicationIdentityClient(endpoint, tokenCredential);
```

## <a name="create-an-identity"></a>Creación de una identidad

Azure Communication Services mantiene un directorio de identidades ligero. Use el método `createUser` para crear una nueva entrada en el directorio con un `Id` único. Almacene la identidad recibida con la asignación a los usuarios de la aplicación. Por ejemplo, almacenándolos en la base de datos del servidor de aplicaciones. La identidad es necesaria más adelante para emitir tokens de acceso.

```javascript
let identityResponse = await identityClient.createUser();
console.log(`\nCreated an identity with ID: ${identityResponse.communicationUserId}`);
```

## <a name="issue-access-tokens"></a>Emitir tokens de acceso

Use el método `getToken` para emitir un token de acceso para la identidad de Communication Services existente. El parámetro `scopes` define un conjunto de primitivas que autorizará este token de acceso. Consulte la [lista de plataformas admitidas](../../concepts/authentication.md). Se puede crear una nueva instancia del parámetro `communicationUser` en función de la representación de cadena de la identidad de Azure Communication Services.

```javascript
// Issue an access token with the "voip" scope for an identity
let tokenResponse = await identityClient.getToken(identityResponse, ["voip"]);
const { token, expiresOn } = tokenResponse;
console.log(`\nIssued an access token with 'voip' scope that expires at ${expiresOn}:`);
console.log(token);
```

Los tokens de acceso son credenciales de corta duración que deben volver a emitirse. No hacerlo puede provocar una interrupción de la experiencia de los usuarios respecto a la aplicación. La propiedad de respuesta `expiresOn` indica la duración del token de acceso.

## <a name="create-an-identity-and-issue-an-access-token-within-the-same-request"></a>Cree una identidad y emita un token de acceso dentro de la misma solicitud.

Use el método `createUserAndToken` para crear una identidad de Communication Services y emitir un token de acceso para ella. El parámetro `scopes` define un conjunto de primitivas que autorizará este token de acceso. Consulte la [lista de plataformas admitidas](../../concepts/authentication.md).

```javascript
// Issue an identity and an access token with the "voip" scope for the new identity
let identityTokenResponse = await identityClient.createUserAndToken(["voip"]);
const { token, expiresOn, user } = identityTokenResponse;
console.log(`\nCreated an identity with ID: ${user.communicationUserId}`);
console.log(`\nIssued an access token with 'voip' scope that expires at ${expiresOn}:`);
console.log(token);
```

## <a name="refresh-access-tokens"></a>Tokens de acceso de actualización

La actualización de los tokens de acceso es tan fácil como llamar a `getToken` con la misma identidad que se usó para emitir los tokens. También debe proporcionar los `scopes` de los tokens actualizados.

```javascript
// // Value of identityResponse represents the Azure Communication Services identity stored during identity creation and then used to issue the tokens being refreshed
let refreshedTokenResponse = await identityClient.getToken(identityResponse, ["voip"]);
```


## <a name="revoke-access-tokens"></a>Revocación de los tokens de acceso

En algunos casos, puede revocar explícitamente los tokens de acceso. Por ejemplo, cuando el usuario de una aplicación cambia la contraseña que usa para autenticarse en el servicio. El método `revokeTokens` invalida todos los tokens de acceso activos emitidos para la identidad.

```javascript
await identityClient.revokeTokens(identityResponse);
console.log(`\nSuccessfully revoked all access tokens for identity with ID: ${identityResponse.communicationUserId}`);
```

## <a name="delete-an-identity"></a>Eliminación de una identidad

La eliminación de una identidad revoca todos los tokens de acceso activos e impide que se emitan tokens de acceso posteriores para la identidad. También quita todo el contenido conservado asociado a la identidad.

```javascript
await identityClient.deleteUser(identityResponse);
console.log(`\nDeleted the identity with ID: ${identityResponse.communicationUserId}`);
```

## <a name="run-the-code"></a>Ejecución del código

Desde un símbolo del sistema de la consola, vaya al directorio que contiene el archivo *issue-access-token.js* y, a continuación, ejecute el comando `node` siguiente para ejecutar la aplicación.

```console
node ./issue-access-token.js
```
