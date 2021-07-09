---
title: 'Tutorial: Implementación de sitios web de Nuxt.js representados por el servidor en Azure Static Web Apps'
description: Genere e implemente sitios dinámicos de Nuxt.js con Azure Static Web Apps.
services: static-web-apps
author: christiannwamba
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 05/08/2020
ms.author: chnwamba
ms.custom: devx-track-js
ms.openlocfilehash: 7654546ba52406aed5659cc736bb9a538a73ac7f
ms.sourcegitcommit: 0ce834cd348bb8b28a5f7f612c2807084cde8e8f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109813243"
---
# <a name="deploy-server-rendered-nuxtjs-websites-on-azure-static-web-apps"></a>Implementación de sitios web de Nuxt.js representados por el servidor en Azure Static Web Apps

En este tutorial, aprenderá a implementar un sitio web estático generado por [Nuxt.js](https://nuxtjs.org) en [Azure Static Web Apps](overview.md). Para empezar, aprenderá a instalar, configurar e implementar una aplicación de Nuxt.js. Durante este proceso, también aprenderá a abordar los desafíos comunes a los que debe enfrentarse a menudo al generar páginas estáticas con Nuxt.js.

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/).
- Una cuenta de GitHub. [Cree una cuenta gratuita](https://github.com/join).
- [Node.js](https://nodejs.org) instalado.

## <a name="set-up-a-nuxtjs-app"></a>Configuración de una aplicación de Nuxt.js

Puede configurar un nuevo proyecto de Nuxt.js mediante `create-nuxt-app`. En lugar de un proyecto nuevo, en este tutorial comenzará clonando un repositorio existente. Este repositorio está configurado para demostrar cómo implementar una aplicación dinámica de Nuxt.js como sitio estático.

1. Cree un repositorio en su cuenta de GitHub desde un repositorio de plantillas.
1. Vaya a [http://github.com/staticwebdev/nuxtjs-starter/generate](https://github.com/login?return_to=/staticwebdev/nuxtjs-starter/generate).
1. Asigne el nombre **nuxtjs-starter** al repositorio.
1. A continuación, clone el nuevo repositorio en la máquina. Asegúrese de reemplazar <YOUR_GITHUB_ACCOUNT_NAME> por el nombre de su cuenta.

    ```bash
    git clone http://github.com/<YOUR_GITHUB_ACCOUNT_NAME>/nuxtjs-starter
    ```

1. Vaya a la aplicación de Nuxt.js recién clonada:

   ```bash
   cd nuxtjs-starter
   ```

1. Instale las dependencias:

    ```bash
    npm install
    ```

1. Inicie la aplicación de Nuxt.js en el entorno de desarrollo:

    ```bash
    npm run dev
    ```

Vaya a `http://localhost:3000` para abrir la aplicación, donde debería ver el siguiente sitio web abierto en el explorador de su preferencia:

:::image type="content" source="media/deploy-nuxtjs/start-nuxtjs-app.png" alt-text="Inicio de la aplicación de Nuxt.js":::

Al hacer clic en un marco/biblioteca, debería ver una página de detalles acerca del elemento seleccionado:

:::image type="content" source="media/deploy-nuxtjs/start-nuxtjs-details.png" alt-text="Página de detalles":::

## <a name="generate-a-static-website-from-nuxtjs-build"></a>Generación de un sitio web estático desde la compilación de Nuxt.js

Al compilar un sitio de Nuxt.js mediante `npm run build`, la aplicación se crea como aplicación web tradicional, no como sitio estático. Para generar un sitio estático, utilice la siguiente configuración de aplicación.

1. Actualice el script de compilación de _package.json_ para únicamente generar un sitio estático con el comando `nuxt generate`:

    ```json
    "scripts": {
      "dev": "nuxt dev",
      "build": "nuxt generate",
    },
    ```

    Ahora, con este comando en contexto, Static Web Apps ejecutará el script `build` cada vez que se inserte una confirmación.

1. Genere un sitio estático:

    ```bash
    npm run build
    ```

    Nuxt.js generará el sitio estático y lo copiará en una carpeta _dist_ en la raíz de su directorio de trabajo.

    > [!NOTE]
    > Esta carpeta aparece en el archivo _.gitignore_ porque debe generarlo CI/CD en el momento de la implementación.

## <a name="push-your-static-website-to-github"></a>Inserción del sitio web estático en GitHub

Azure Static Web Apps implementa la aplicación desde un repositorio de GitHub y lo sigue haciendo para cada confirmación insertada en una rama designada. Utilice los siguientes comandos para sincronizar los cambios en GitHub.

1. Agregue al "stage" todos los archivos cambiados:

    ```bash
    git add .
    ```

1. Confirme todos los cambios.

    ```bash
    git commit -m "Update build config"
    ```

1. Envíe los cambios a GitHub.

    ```bash
    git push origin main
    ```

## <a name="deploy-your-static-website"></a>Implementación del sitio web estático

En los pasos siguientes se muestra cómo vincular la aplicación que acaba de insertar en GitHub con Azure Static Web Apps. Una vez que esté en Azure, podrá implementar la aplicación en un entorno de producción.

### <a name="create-an-azure-static-web-apps-resource"></a>Creación de un recurso de Azure Static Web Apps

1. Acceda a [Azure Portal](https://portal.azure.com).
1. Seleccione **Crear un recurso**.
1. Busque **Static Web Apps**.
1. Seleccione **Static Web Apps**.
1. Seleccione **Crear**.
1. En la pestaña _Datos básicos_, especifique los valores siguientes.

    | Propiedad | Valor |
    | --- | --- |
    | _Suscripción_ | El nombre de la suscripción de Azure. |
    | _Grupos de recursos_ | **my-nuxtjs-group**  |
    | _Nombre_ | **my-nuxtjs-app** |
    | _Tipo de plan_ | **Gratis** |
    | _Región para la API y los entornos de ensayo de Azure Functions_ | Seleccione la región más cercana a la suya. |
    | _Origen_ | **GitHub** |

1. Seleccione **Iniciar sesión con GitHub** y autentíquese con GitHub.

1. Escriba los siguientes valores de GitHub.

    | Propiedad | Valor |
    | --- | --- |
    | _Organización_ | Seleccione la organización de GitHub que quiera. |
    | _Repositorio_ | Seleccione el repositorio que ha creado anteriormente. |
    | _Rama_ | Seleccione **main** (principal). |

1. En la sección _Detalles de la compilación_, seleccione **Personalizado** en la lista desplegable _Valores preestablecidos de compilación_ y conserve los valores predeterminados.

1. En _Ubicación de la aplicación_, escriba **./** en el cuadro.
1. Deje en blanco el cuadro _Ubicación de la aplicación_.
1. En el cuadro _Ubicación del resultado_, escriba **fuera**.

### <a name="review-and-create"></a>Revisar y crear

1. Seleccione el botón **Revisar y crear** para comprobar que todos los detalles sean correctos.

1. Seleccione **Crear** para comenzar la creación de la aplicación web estática de App Service y aprovisionar una Acción de GitHub para la implementación.

1. Cuando se complete la implementación, haga clic en **Ir al recurso**.

1. En la ventana _Información general_, haga clic en el vínculo *Dirección URL* para abrir la aplicación implementada.

Si el sitio web no se carga de inmediato, el flujo de trabajo de Acciones de GitHub en segundo plano sigue en ejecución. Una vez que se complete el flujo de trabajo, puede actualizar el explorador para ver la aplicación web.

Para comprobar el estado de los flujos de trabajo de Acciones, navegue a Acciones del repositorio:

```url
https://github.com/<YOUR_GITHUB_USERNAME>/nuxtjs-starter/actions
```

### <a name="sync-changes"></a>Sincronización de cambios

Al crear la aplicación, Azure Static Web Apps creó un archivo de flujo de trabajo de Acciones de GitHub en el repositorio. Debe poner este archivo en el repositorio local para que el historial de Git se sincronice.

Vuelva al terminal y ejecute el siguiente comando: `git pull origin main`.

## <a name="configure-dynamic-routes"></a>Configuración de rutas dinámicas

Vaya al sitio recién implementado y haga clic en uno de los logotipos de marco o biblioteca. En lugar de obtener una página de detalles, aparecerá una página de error 404.

:::image type="content" source="media/deploy-nuxtjs/404-in-production.png" alt-text="404 en rutas dinámicas":::

El motivo es que Nuxt.js generó el sitio estático, pero solo lo hizo para la página principal. Nuxt.js puede generar archivos `.html` estáticos equivalentes para cada archivo de páginas `.vue`, pero hay una excepción. 

Si se trata de una página dinámica, por ejemplo `_id.vue`, no tendrá suficiente información para generar un HTML estático desde esa página dinámica. Tendrá que proporcionar de manear explícita las rutas de acceso posibles para las rutas dinámicas.

## <a name="generate-static-pages-from-dynamic-routes"></a>Generación de páginas estáticas a partir de rutas dinámicas

1. Actualice el archivo _nuxt.config.js_ para que Nuxt.js use una lista de todos los datos disponibles para generar páginas estáticas para cada marco o biblioteca:

   ```javascript
     import { projects } from "./utils/projectsData";

     export default {
       mode: "universal",

       //...truncated

       generate: {
         async routes() {
           const paths = [];

           projects.forEach(project => {
             paths.push(`/project/${project.slug}`);
           });

           return paths;
         }
       }
     };
   ```

   > [!NOTE]
   > `routes` es una función asincrónica, por lo que puede hacer una solicitud a una API en esta función y usar la lista devuelta para generar las rutas de acceso.

2. Envíe los nuevos cambios al repositorio de GitHub y espere unos minutos mientras las Acciones de GitHub compilan de nuevo el sitio. Una vez que se complete la compilación, el error 404 desaparecerá.

   :::image type="content" source="media/deploy-nuxtjs/404-in-production-fixed.png" alt-text="404 en rutas dinámicas corregido":::

> [!div class="nextstepaction"]
> [Configuración de un dominio personalizado](custom-domain.md)
