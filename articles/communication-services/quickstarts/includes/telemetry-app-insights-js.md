---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: orwatson
manager: vravikumar
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 06/01/2021
ms.topic: include
ms.custom: include file
ms.author: orwatson
ms.openlocfilehash: f044b23d26bee71fc3f6a28f96a42c42a01f4d02
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111593143"
---
## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Versiones Active LTS y Maintenance LTS de [Node.js](https://nodejs.org/).
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../create-communication-resource.md).
- Cree un [recurso de Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/create-new-resource) en Azure Portal.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-nodejs-app"></a>Creación de una nueva aplicación Node.js

Si ya tiene una aplicación que usa una biblioteca de ACS y se siente cómodo con JavaScript, no dude en ir directamente a [Configuración del seguimiento](#setup-tracer).

Para empezar, necesitará una aplicación de JS que use cualquiera de nuestras bibliotecas cliente. Usaremos la biblioteca [@azure/communication-identity](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/communication/communication-identity) para crear la aplicación. Siga los pasos a continuación para configurar la aplicación de ejemplo:

Abra la ventana de comandos o el terminal, cree un nuevo directorio y navegue hasta él.

```console
mkdir export-js-telemetry && cd export-js-telemetry
```
Ejecute `npm init -y` para crear un archivo **package.json** con la configuración predeterminada.

```console
npm init -y
```

### <a name="install-dependencies"></a>Instalar dependencias

Use el comando `npm install` para instalar las dependencias que usará la aplicación.

```bash
npm install @azure/communication-identity @azure/monitor-opentelemetry-exporter @opentelemetry/node@^0.14.0 @opentelemetry/plugins-node-core --save
```

La opción `--save` agrega las bibliotecas como dependencias en el archivo **package.json** creado en el paso anterior.

Es importante que no olvide la versión específica de `@opentelemetry/node@^0.14.0`. En el momento de escribir esta guía, esta es la versión más reciente del paquete que era totalmente compatible con `@azure/monitor-opentelemetry-exporter`.

### <a name="add-main-app-file"></a>Adición del archivo de aplicación principal

Cree un archivo denominado **token.js** en la raíz del directorio. Agregue el código siguiente a él:

```javascript
// token.js

const { CommunicationIdentityClient } = require("@azure/communication-identity");

async function main() {
  const ACS_CONNECTION_STRING = "<your-acs-connection-string>"
  const client = new CommunicationIdentityClient(ACS_CONNECTION_STRING);

  console.log("Creating user...")
  const user = await client.createUser();
  console.log(`User ${user.communicationUserId} was created successfully.\n`);

  console.log(`Issuing token for ${user.communicationUserId}...`);
  const { token } = await client.getToken(user, ["chat"]);
  console.log("Token issued successfully.\n")
  console.log(`${token}\n`);

  console.log(`Revoking token for ${user.communicationUserId}...`);
  await client.revokeTokens(user);
  console.log("Token revoked successfully.\n");

  console.log(`Deleting user ${user.communicationUserId}...`);
  await client.deleteUser(user);
  console.log("User deleted successfully.\n");
}

main().catch((error) => {
  console.log("Encountered an error:", error);
  process.exit(1);
});
```
## <a name="setup-tracer"></a>Configuración del seguimiento

Cree un archivo denominado **tracing.js** en la raíz del directorio. Agregue el código siguiente a él:

```javascript
// tracing.js

const azureSdkTracing = require("@azure/core-tracing");
const { AzureMonitorTraceExporter } = require("@azure/monitor-opentelemetry-exporter");
const { NodeTracerProvider } = require("@opentelemetry/node");
const { BatchSpanProcessor } = require("@opentelemetry/tracing");

const AI_CONNECTION_STRING = "<your-application-insights-connection-string>";
const provider = new NodeTracerProvider();
const azExporter = new AzureMonitorTraceExporter({
  connectionString: AI_CONNECTION_STRING
});

provider.addSpanProcessor(
  new BatchSpanProcessor(azExporter, {
    bufferSize: 1000, // 1000 spans
    bufferTimeout: 5000 // 5 seconds
  })
);

provider.register();

const tracer = provider.getTracer("sample tracer");
azureSdkTracing.setTracer(tracer);

exports.default = tracer;
```

### <a name="add-tracer-to-main-app"></a>Adición del seguimiento a la aplicación principal

Actualice el archivo **token.js** de la manera siguiente:

Aplique require a **tracing.js**:

```javascript
const { CommunicationIdentityClient } = require("@azure/communication-identity");
const tracer = require("./tracing.js");

async function main() {
```

Cree rootSpan y exporte los datos:

```javascript
  console.log("User deleted successfully.\n");
}

const rootSpan = tracer.startSpan("Root Identity Span");
tracer.withSpan(rootSpan, async function() {
  try {
    await main();
  } catch (error) {
    console.error("Error running sample:", error);
  } finally {
    // End the optional root span on completion
    rootSpan.end();
  }
}).then(function() {
  console.log("Awaiting batched span processor to export batched spans...");

  setTimeout(function() {
    console.log("Spans exported.");
  }, 6000);
});
```

## <a name="run-the-code"></a>Ejecución del código

Asegúrese de reemplazar los textos de los marcadores de posición por valores de cadena de conexión válidos.

Use el comando node para ejecutar el código que agregó al archivo **token.js**.

```console
node token.js
```