---
title: archivo de inclusión
description: archivo de inclusión
services: service-bus-relay
author: clemensv
ms.service: service-bus-relay
ms.topic: include
ms.date: 05/02/2018
ms.author: clemensv
ms.custom: include file
ms.openlocfilehash: 3c18efa7eb520b765c9bb3c2aff00104f971f5a8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186553"
---
### <a name="create-a-nodejs-application"></a>Creación de una aplicación Node.js

Si ha desactivado la opción que indica que se requiere la autorización del cliente al crear la retransmisión, puede enviar solicitudes a la dirección URL de conexiones híbridas con un explorador. Para acceder a los puntos de conexión protegidos, debe crear y pasar un token en el encabezado `ServiceBusAuthorization`, que se muestra aquí.

Para comenzar, cree un nuevo archivo JavaScript denominado `sender.js`.

### <a name="add-the-relay-npm-package"></a>Adición del paquete Relay NPM

Ejecute `npm install hyco-https` desde un símbolo del sistema del nodo en la carpeta del proyecto. Este paquete también importa el paquete `https` normal. En el cliente, la principal diferencia es que el paquete proporciona funciones para construir identificadores URI de retransmisión y tokens.

### <a name="write-some-code-to-send-messages"></a>Escritura de código para enviar mensajes

1. Agregue lo siguiente `constants` en la parte superior del archivo `sender.js`.
   
    ```js
    const https = require('hyco-https');
    ```

2. Agregue la siguientes constantes al archivo `sender.js` para los detalles de la conexión híbrida. Reemplace los marcadores de posición entre corchetes por los valores que obtuvo al crear la conexión híbrida.
   
   1. `const ns`: el espacio de nombres de Relay. Asegúrese de utilizar el nombre de espacio de nombres completo; por ejemplo, `{namespace}.servicebus.windows.net`.
   2. `const path`: el nombre de la conexión híbrida.
   3. `const keyrule`: el nombre de la clave SAS.
   4. `const key`: el valor de la clave SAS.

3. Agregue el siguiente código al archivo `sender.js`. Observará que el código no difiere significativamente del uso normal del cliente HTTPS de Node.js; simplemente agrega el encabezado de autorización.
   
    ```js
   https.get({
        hostname : ns,
        path : (!path || path.length == 0 || path[0] !== '/'?'/':'') + path,
        port : 443,
        headers : {
            'ServiceBusAuthorization' : 
                https.createRelayToken(https.createRelayHttpsUri(ns, path), keyrule, key)
        }
    }, (res) => {
        let error;
        if (res.statusCode !== 200) {
            console.error('Request Failed.\n Status Code: ${statusCode}');
            res.resume();
        } 
        else {
            res.setEncoding('utf8');
            res.on('data', (chunk) => {
                console.log(`BODY: ${chunk}`);
            });
            res.on('end', () => {
                console.log('No more data in response.');
            });
        };
    }).on('error', (e) => {
        console.error(`Got error: ${e.message}`);
    });
    ```
    Este es el aspecto que debería tener el archivo sender.js:
   
    ```js
    const https = require('hyco-https');
       
    const ns = "{RelayNamespace}";
    const path = "{HybridConnectionName}";
    const keyrule = "{SASKeyName}";
    const key = "{SASKeyValue}";
   
    https.get({
        hostname : ns,
        path : (!path || path.length == 0 || path[0] !== '/'?'/':'') + path,
        port : 443,
        headers : {
            'ServiceBusAuthorization' : 
                https.createRelayToken(https.createRelayHttpsUri(ns, path), keyrule, key)
        }
    }, (res) => {
        let error;
        if (res.statusCode !== 200) {
            console.error('Request Failed.\n Status Code: ${statusCode}');
            res.resume();
        } 
        else {
            res.setEncoding('utf8');
            res.on('data', (chunk) => {
                console.log(`BODY: ${chunk}`);
            });
            res.on('end', () => {
                console.log('No more data in response.');
            });
        };
    }).on('error', (e) => {
        console.error(`Got error: ${e.message}`);
    });
    ```

