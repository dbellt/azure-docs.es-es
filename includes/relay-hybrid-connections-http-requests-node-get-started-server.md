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
ms.openlocfilehash: fdd4e26839661ab9765b7d496e7f60c3686ba637
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186552"
---
### <a name="create-a-nodejs-application"></a>Creación de una aplicación Node.js

Cree un nuevo archivo JavaScript denominado `listener.js`.

### <a name="add-the-relay-npm-package"></a>Adición del paquete Relay NPM

Ejecute `npm install hyco-https` desde un símbolo del sistema del nodo en la carpeta del proyecto.

### <a name="write-some-code-to-handle-requests"></a>Escritura de código para administrar las solicitudes

1. Agregue la siguiente constante al principio del archivo `listener.js`.

    ```js
    const https = require('hyco-https');
    ```
2. Agregue la siguientes constantes al archivo `listener.js` para los detalles de la conexión híbrida. Reemplace los marcadores de posición entre corchetes por los valores que obtuvo al crear la conexión híbrida.

   1. `const ns`: el espacio de nombres de Relay. Asegúrese de utilizar el nombre de espacio de nombres completo; por ejemplo, `{namespace}.servicebus.windows.net`.
   2. `const path`: el nombre de la conexión híbrida.
   3. `const keyrule`: el nombre de la clave SAS.
   4. `const key`: el valor de la clave SAS.

3. Agregue el siguiente código al archivo `listener.js`. :

    Observará que el código no es muy diferente de cualquier ejemplo sencillo de servidor HTTP que puede encontrar en los tutoriales para principiantes de Node.js, a excepción del uso de `createRelayedServer` en lugar de la típica función `createServer`.

    ```js
    var uri = https.createRelayListenUri(ns, path);
    var server = https.createRelayedServer(
        {
            server : uri,
            token : () => https.createRelayToken(uri, keyrule, key)
        },
        (req, res) => {
            console.log('request accepted: ' + req.method + ' on ' + req.url);
            res.setHeader('Content-Type', 'text/html');
            res.end('<html><head><title>Hey!</title></head><body>Relayed Node.js Server!</body></html>');
        });

    server.listen( (err) => {
            if (err) {
              return console.log('something bad happened', err)
            }
            console.log(`server is listening on ${port}`)
          });

    server.on('error', (err) => {
        console.log('error: ' + err);
    });
    ```
    Este es el aspecto que debería tener el archivo listener.cs:
   
    ```js
    const https = require('hyco-https');
   
    const ns = "{RelayNamespace}";
    const path = "{HybridConnectionName}";
    const keyrule = "{SASKeyName}";
    const key = "{SASKeyValue}";
   
    var uri = https.createRelayListenUri(ns, path);
    var server = https.createRelayedServer(
        {
            server : uri,
            token : () => https.createRelayToken(uri, keyrule, key)
        },
        (req, res) => {
            console.log('request accepted: ' + req.method + ' on ' + req.url);
            res.setHeader('Content-Type', 'text/html');
            res.end('<html><head><title>Hey!</title></head><body>Relayed Node.js Server!</body></html>');
        });

    server.listen( (err) => {
            if (err) {
              return console.log('something bad happened', err)
            }
            console.log(`server is listening on ${port}`)
          });

    server.on('error', (err) => {
        console.log('error: ' + err);
    });
    ```

