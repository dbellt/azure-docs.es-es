---
title: 'Tutorial: Publicación de un sitio de VuePress en Azure Static Web Apps'
description: En este tutorial se muestra cómo implementar una aplicación de VuePress en Azure Static Web Apps.
services: static-web-apps
author: aaronpowell
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 05/11/2021
ms.author: aapowell
ms.custom: devx-track-js
ms.openlocfilehash: 8b681816feb9f28b90c71e924681a7e787d52594
ms.sourcegitcommit: 0ce834cd348bb8b28a5f7f612c2807084cde8e8f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109814395"
---
# <a name="tutorial-publish-a-vuepress-site-to-azure-static-web-apps"></a>Tutorial: Publicación de un sitio de VuePress en Azure Static Web Apps

En este artículo se muestra cómo crear e implementar una aplicación web de [VuePress](https://vuepress.vuejs.org/) en [Azure Static Web Apps](overview.md). El resultado final es una nueva aplicación de Azure Static Web Apps, con las Acciones de GitHub asociadas, que le da control sobre cómo se compila y publica la aplicación.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> - Crear una aplicación de VuePress
> - Configurar Azure Static Web Apps
> - Implementar la aplicación de VuePress en Azure

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. En caso de no tener una, puede [crear una cuenta gratuita](https://azure.microsoft.com/free/).
- Una cuenta de GitHub. En caso de no tener una, puede [crear una cuenta gratuita](https://github.com/join).
- [Node.js](https://nodejs.org) instalado.

## <a name="create-a-vuepress-app"></a>Creación de una aplicación de VuePress

Cree una aplicación de VuePress desde la interfaz de la línea de comandos (CLI):

1. Cree una carpeta para la aplicación de VuePress.

   ```bash
   mkdir static-site
   ```

1. Agregue un archivo _README.md_ a la carpeta.

   ```bash
   echo '# Hello From VuePress' > README.md
   ```

1. Inicialice el archivo _package.json_.

   ```bash
   npm init -y
   ```

1. Agregue VuePress como `devDependency`.

   ```bash
   npm install --save-dev vuepress
   ```

1. Abra el archivo _package.json_ en un editor de texto y agregue un comando de compilación a la sección [`scripts`](https://docs.npmjs.com/cli-commands/run-script.html).

   ```json
   ...
   "scripts": {
       "build": "vuepress build"
   }
   ...
   ```

1. Cree un archivo _.gitignore_ para excluir la carpeta _node\_modules_.

    ```bash
    echo 'node_modules' > .gitignore
    ```

1. Inicialice un repositorio de Git.

   ```bash
    git init
    git add -A
    git commit -m "initial commit"
   ```

## <a name="push-your-application-to-github"></a>Inserción de la aplicación en GitHub

Necesita un repositorio en GitHub para conectarse a Azure Static Web Apps. En los pasos siguientes se muestra cómo crear un repositorio para el sitio.

1. Cree un repositorio de GitHub en blanco (no cree un archivo Léame) desde [https://github.com/new](https://github.com/new), denominado **vuepress-static-app**.

1. Agregue el repositorio de GitHub como remoto al repositorio local. Asegúrese de agregar el nombre de usuario de GitHub en lugar del marcador de posición `<YOUR_USER_NAME>` en el comando siguiente.

   ```bash
   git remote add origin https://github.com/<YOUR_USER_NAME>/vuepress-static-app
   ```

1. Inserte el repositorio local en GitHub.

   ```bash
   git push --set-upstream origin main
   ```

## <a name="deploy-your-web-app"></a>Implementación de la aplicación web

En los pasos siguientes se muestra cómo crear una aplicación de sitio estática e implementarla en un entorno de producción.

### <a name="create-the-application"></a>Creación de la aplicación

1. Vaya a [Azure Portal](https://portal.azure.com).
1. Seleccione **Crear un recurso**
1. Busque **Static Web Apps**.
1. Seleccione **Static Web Apps**.
1. Seleccione **Crear**
1. En la pestaña _Datos básicos_, especifique los valores siguientes.

    | Propiedad | Valor |
    | --- | --- |
    | _Suscripción_ | El nombre de la suscripción de Azure. |
    | _Grupos de recursos_ | **my-vuepress-group**  |
    | _Nombre_ | **vuepress-static-app** |
    | _Tipo de plan_ | **Gratis** |
    | _Región para la API y los entornos de ensayo de Azure Functions_ | Seleccione la región más cercana a la suya. |
    | _Origen_ | **GitHub** |

1. Seleccione **Iniciar sesión con GitHub** y autentíquese con GitHub.

1. Escriba los siguientes valores de GitHub.

    | Propiedad | Valor |
    | --- | --- |
    | _Organización_ | Seleccione la organización de GitHub que quiera. |
    | _Repositorio_ | Seleccione **vuepress-static-app**. |
    | _Rama_ | Seleccione **main** (principal). |

1. En la sección _Detalles de la compilación_, seleccione **VuePress** en la lista desplegable _Valores preestablecidos de compilación_ y conserve los valores predeterminados.

### <a name="review-and-create"></a>Revisar y crear

1. Seleccione el botón **Revisar y crear** para comprobar que todos los detalles sean correctos.

1. Seleccione **Crear** para comenzar la creación de la aplicación web estática de App Service y aprovisionar una Acción de GitHub para la implementación.

1. Cuando se complete la implementación, haga clic en **Ir al recurso**.

1. En la pantalla del recurso, haga clic en el vínculo _Dirección URL_ para abrir la aplicación implementada. Es posible que tenga que esperar uno o dos minutos para que se complete la Acción de GitHub.

   :::image type="content" source="./media/publish-vuepress/deployed-app.png" alt-text="Aplicación implementada":::

### <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cleanup-resource](../../includes/static-web-apps-cleanup-resource.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Agregar un dominio personalizado](custom-domain.md)
