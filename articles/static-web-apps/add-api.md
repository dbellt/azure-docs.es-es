---
title: Incorporación de una API en Azure Static Web Apps con Azure Functions
description: Para empezar a usar Azure Static Web Apps, agregue una API sin servidor a la aplicación web estática mediante Azure Functions.
services: static-web-apps
author: manekinekko
ms.service: static-web-apps
ms.topic: how-to
ms.date: 05/14/2021
ms.author: wachegha
ms.custom: devx-track-js
ms.openlocfilehash: 939c63edba204ff903a8616eef1db5e031397a3f
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110066185"
---
# <a name="add-an-api-to-azure-static-web-apps-with-azure-functions"></a>Incorporación de una API en Azure Static Web Apps con Azure Functions

Puede agregar API sin servidor a Azure Static Web Apps con tecnología de Azure Functions. En este artículo se muestra cómo agregar e implementar una API en un sitio de Azure Static Web Apps.

> [!NOTE]
> Las funciones proporcionadas de forma predeterminada en Static Web Apps están preconfiguradas para proporcionar puntos de conexión de API seguros y solo admiten funciones desencadenadas por HTTP. Consulte [Compatibilidad de API con Azure Functions](apis.md) para obtener información sobre cómo difieren de las aplicaciones de Azure Functions independientes.

## <a name="prerequisites"></a>Requisitos previos

- Cuenta de Azure con una suscripción activa.
  - Si no tiene una cuenta, puede [crear una gratis](https://azure.microsoft.com/free).
- [Visual Studio Code](https://code.visualstudio.com/)
- [Extensión de Azure Static Web Apps](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestaticwebapps) para Visual Studio Code
- [Node.js](https://nodejs.org/download/) para ejecutar la aplicación de front-end y la API

## <a name="create-the-static-web-app"></a>Creación de la aplicación web estática

Antes de agregar una API, cree e implemente una aplicación de front-end en Azure Static Web Apps. Use una aplicación existente que ya haya implementado o cree una siguiendo las instrucciones de [Inicio rápido: creación del primer sitio estático con Azure Static Web Apps](getting-started.md).

En Visual Studio Code, abra la raíz del repositorio de la aplicación. La estructura de carpetas contiene el origen de la aplicación de front-end y el flujo de trabajo de GitHub de Static Web Apps en la carpeta _.github/workflows_.

```Files
├── .github
│   └── workflows
│       └── azure-static-web-apps-<DEFAULT_HOSTNAME>.yml
│
└── (folders and files from your static web app)
```

## <a name="create-the-api"></a>Creación de la API

Cree un proyecto de Azure Functions para la API de la aplicación web estática. De forma predeterminada, la extensión de Visual Studio Code de Static Web Apps crea el proyecto en una carpeta denominada _api_ en la raíz del repositorio.

1. Presione <kbd>F1</kbd> para abrir la paleta de comandos.

1. Seleccione **Azure Static Web Apps: Create HTTP Function** (Azure Static Web Apps: crear una función HTTP). Si se le pide que instale la extensión de Azure Functions, instálela y vuelva a ejecutar este comando.

1. Cuando se le solicite, escriba los siguientes valores:

    | Prompt | Valor |
    | --- | --- |
    | Selección de un idioma | **JavaScript** |
    | Proporcionar un nombre de función | **message** |

    Un proyecto de Azure Functions se genera con una función desencadenada por HTTP. Ahora la aplicación tiene una estructura de proyecto similar a la del ejemplo siguiente.

    ```Files
    ├── .github
    │   └── workflows
    │       └── azure-static-web-apps-<DEFAULT_HOSTNAME>.yml
    │
    ├── api
    │   ├── message
    │   │   ├── function.json
    │   │   └── index.js
    │   ├── host.json
    │   ├── local.settings.json
    │   └── package.json
    │
    └── (folders and files from your static web app)
    ```

1. A continuación, cambie la función `message` para devolver un mensaje al front-end. Actualice la función de _api/message/index.js_  con el código siguiente.

    ```javascript
    module.exports = async function (context, req) {
        context.res.json({
            text: "Hello from the API"
        });
    };
    ```

> [!TIP]
> Para agregar más funciones de API, puede volver a ejecutar el comando **Azure Static Web Apps: Create HTTP Function** (Azure Static Web Apps: crear una función HTTP).

## <a name="update-the-frontend-app-to-call-the-api"></a>Actualización de la aplicación de front-end para llamar a la API

Actualice la aplicación de front-end para llamar a la API en `/api/message` y mostrar el mensaje de respuesta.

Si ha usado los inicios rápidos para crear la aplicación, siga estas instrucciones para aplicar las actualizaciones.

# <a name="no-framework"></a>[Ningún marco](#tab/vanilla-javascript)

Actualice el contenido del archivo _index.html_ con el código siguiente para capturar el texto de la función de la API y mostrarlo en la pantalla.

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="styles.css">
    <title>Vanilla JavaScript App</title>
</head>

<body>
    <main>
    <h1>Vanilla JavaScript App</h1>
    <p>Loading content from the API: <b id="name">...</b></p>
    </main>

    <script>
    (async function() {
        const { text } = await( await fetch(`/api/message`)).json();
        document.querySelector('#name').textContent = text;
    }())
    </script>
</body>

</html>
```

# <a name="angular"></a>[Angular](#tab/angular)

1. Actualice el contenido de _src/app/app.module.ts_ con el código siguiente para habilitar `HttpClient` en la aplicación.

    ```typescript
    import { BrowserModule } from "@angular/platform-browser";
    import { NgModule } from "@angular/core";
    import { HttpClientModule } from '@angular/common/http';
    
    import { AppComponent } from "./app.component";
    
    @NgModule({
      declarations: [AppComponent],
      imports: [BrowserModule, HttpClientModule],
      bootstrap: [AppComponent]
    })
    export class AppModule {}
    ```

1. A continuación, actualice el contenido del archivo _src/app/app.component.ts_ con el código siguiente para capturar el texto de la función de la API y mostrarlo en la pantalla.

    ```typescript
    import { HttpClient } from '@angular/common/http';
    import { Component } from '@angular/core';
    
    @Component({
      selector: 'app-root',
      template: `<div>{{message}}</div>`,
    })
    export class AppComponent {
      message = '';
    
      constructor(private http: HttpClient) {
        this.http.get('/api/message')
          .subscribe((resp: any) => this.message = resp.text);
      }
    }
    ```

# <a name="react"></a>[React](#tab/react)

Actualice el contenido del archivo _src/App.js_ con el código siguiente para capturar el texto de la función de la API y mostrarlo en la pantalla.

```javascript
import React, { useState, useEffect } from 'react';

function App() {
  const [data, setData] = useState('');

  useEffect(() => {
    (async function () {
      const { text } = await( await fetch(`/api/message`)).json();
      setData(text);
    })();
  });

  return <div>{data}</div>;
}

export default App;
```

# <a name="vue"></a>[Vue](#tab/vue)

Actualice el contenido del archivo _src/App.vue_ con el código siguiente para capturar el texto de la función de la API y mostrarlo en la pantalla.

```javascript
<template>
  <div>{{ message }}</div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      message: ""
    };
  },
  async mounted() {
    const { text } = await (await fetch("/api/message")).json();
    this.message = text;
  }
};
</script>
```

---

## <a name="run-the-frontend-and-api-locally"></a>Ejecución local del front-end y la API

Para ejecutar localmente la aplicación de front-end y la API juntas, Azure Static Web Apps proporciona una CLI que emula el entorno en la nube. La CLI utiliza Azure Functions Core Tools para ejecutar la API.

### <a name="install-command-line-tools"></a>Instalación de herramientas de línea de comandos

Asegúrese de que tiene instaladas las herramientas de línea de comandos necesarias.

1. Instale la CLI de Azure Static Web Apps.
    ```bash
    npm install -g @azure/static-web-apps-cli
    ```

1. Instale Azure Functions Core Tools v3.
    ```bash
    npm install -g azure-functions-core-tools@3
    ```

### <a name="build-frontend-app"></a>Compilación de una aplicación de front-end

Si la aplicación usa un marco, compile la aplicación para generar la salida antes de ejecutar la CLI de Static Web Apps.

# <a name="no-framework"></a>[Ningún marco](#tab/vanilla-javascript)

No es necesario compilar la aplicación.

# <a name="angular"></a>[Angular](#tab/angular)

Compile la aplicación en la carpeta _dist/angular-basic_.

```bash
npm run build --prod
```

# <a name="react"></a>[React](#tab/react)

Compile la aplicación en la carpeta _build_.

```bash
npm run build
```

# <a name="vue"></a>[Vue](#tab/vue)

Compile la aplicación en la carpeta _dist_.

```bash
npm run build
```

---

### <a name="start-the-cli"></a>Iniciar la CLI

Para ejecutar la aplicación de front-end y la API juntas, inicie la aplicación con la CLI de Static Web Apps. La ejecución de las dos partes de la aplicación de esta manera permite que la CLI sirva la salida de compilación del front-end desde una carpeta y hace que la API sea accesible para la aplicación en ejecución.

1. En la raíz del repositorio, inicie la CLI de Static Web Apps con el comando `start`. Ajuste los argumentos si la aplicación tiene una estructura de carpetas diferente.

    # <a name="no-framework"></a>[Ningún marco](#tab/vanilla-javascript)

    Pase la carpeta actual (`.`) y la carpeta de la API (`api`) a la CLI.
     
    ```bash
    swa start . --api api
    ```

    # <a name="angular"></a>[Angular](#tab/angular)

    Pase la carpeta de la salida de compilación (`dist/angular-basic`) y la carpeta de la API (`api`) a la CLI.

    ```bash
    swa start dist/angular-basic --api api
    ```

    # <a name="react"></a>[React](#tab/react)

    Pase la carpeta de la salida de compilación (`build`) y la carpeta de la API (`api`) a la CLI.

    ```bash
    swa start build --api api
    ```

    # <a name="vue"></a>[Vue](#tab/vue)

    Pase la carpeta de la salida de compilación (`dist`) y la carpeta de la API (`api`) a la CLI.

    ```bash
    swa start dist --api api
    ```

    ---

1. Cuando se inicien los procesos de la CLI, acceda a la aplicación en `http://localhost:4280/`. Observe cómo la página llama a la API y muestra su salida, `Hello from the API`.

1. Para detener la CLI, escriba <kbd>Ctrl-C</kbd>.

## <a name="add-api-location-to-workflow"></a>Adición de la ubicación de la API al flujo de trabajo

Para poder implementar la aplicación en Azure, actualice el flujo de trabajo Acciones de GitHub del repositorio con la ubicación correcta de la carpeta de la API.

1. Abra el flujo de trabajo en _.github/workflows/azure-static-web-apps-\<DEFAULT-HOSTNAME>.yml_.

1. Busque la propiedad `api_location` y establezca el valor en `api`.
1. Guarde el archivo.

## <a name="deploy-changes"></a>Implementación de cambios

Para publicar cambios en la aplicación web estática en Azure, confirme e inserte el código en el repositorio remoto de GitHub.

1. Presione <kbd>F1</kbd> para abrir la paleta de comandos.

1. Seleccione el comando **Git: Confirmar todo**.

1. Cuando se le pida un mensaje de confirmación, escriba **add API** y confirme todos los cambios en el repositorio de Git local.

1. Presione <kbd>F1</kbd> para abrir la paleta de comandos.

1. Seleccione el comando **Git: push**.

    Los cambios se insertan en el repositorio remoto de GitHub, lo que desencadena el flujo de trabajo Acciones de GitHub de Static Web Apps para compilar e implementar la aplicación.

1. Abra el repositorio en GitHub para supervisar el estado de la ejecución del flujo de trabajo.

1. Cuando se complete la ejecución del flujo de trabajo, visite la aplicación web estática para ver los cambios.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración de aplicaciones](./application-settings.md)
