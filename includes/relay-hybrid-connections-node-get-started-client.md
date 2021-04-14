---
author: clemensv
ms.service: service-bus-relay
ms.topic: include
ms.date: 11/09/2018
ms.author: clemensv
ms.openlocfilehash: 9d4f7faa18ee7fae158afb42b8c42287e61dd103
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186902"
---
### <a name="create-a-nodejs-application"></a>Creación de una aplicación Node.js

Cree un nuevo archivo JavaScript denominado `sender.js`.

### <a name="add-the-relay-npm-package"></a>Adición del paquete Relay NPM

Ejecute `npm install hyco-ws` desde un símbolo del sistema del nodo en la carpeta del proyecto.

### <a name="write-some-code-to-send-messages"></a>Escritura de código para enviar mensajes

1. Agregue lo siguiente `constants` en la parte superior del archivo `sender.js`.
   
    ```js
    const WebSocket = require('hyco-ws');
    const readline = require('readline')
        .createInterface({
            input: process.stdin,
            output: process.stdout
        });;
    ```
2. Agregue la siguientes constantes al archivo `sender.js` para los detalles de la conexión híbrida. Reemplace los marcadores de posición entre corchetes por los valores que obtuvo al crear la conexión híbrida.
   
   1. `const ns`: el espacio de nombres de Relay. Asegúrese de utilizar el nombre de espacio de nombres completo; por ejemplo, `{namespace}.servicebus.windows.net`.
   2. `const path`: el nombre de la conexión híbrida.
   3. `const keyrule`: el nombre de la clave SAS.
   4. `const key`: el valor de la clave SAS.

3. Agregue el siguiente código al archivo `sender.js`:
   
    ```js
    WebSocket.relayedConnect(
        WebSocket.createRelaySendUri(ns, path),
        WebSocket.createRelayToken('http://'+ns, keyrule, key),
        function (wss) {
            readline.on('line', (input) => {
                wss.send(input, null);
            });
   
            console.log('Started client interval.');
            wss.on('close', function () {
                console.log('stopping client interval');
                process.exit();
            });
        }
    );
    ```
    Este es el aspecto que debería tener el archivo sender.js:
   
    ```js
    const WebSocket = require('hyco-ws');
    const readline = require('readline')
        .createInterface({
            input: process.stdin,
            output: process.stdout
        });;
   
    const ns = "{RelayNamespace}";
    const path = "{HybridConnectionName}";
    const keyrule = "{SASKeyName}";
    const key = "{SASKeyValue}";
   
    WebSocket.relayedConnect(
        WebSocket.createRelaySendUri(ns, path),
        WebSocket.createRelayToken('http://'+ns, keyrule, key),
        function (wss) {
            readline.on('line', (input) => {
                wss.send(input, null);
            });
   
            console.log('Started client interval.');
            wss.on('close', function () {
                console.log('stopping client interval');
                process.exit();
            });
        }
    );
    ```

