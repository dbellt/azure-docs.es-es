---
title: 'Tutorial: Publicación de un sitio de Hugo en Azure Static Web Apps'
description: Obtenga información sobre cómo implementar una aplicación de Hugo en Azure Static Web Apps.
services: static-web-apps
author: aaronpowell
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 05/11/2021
ms.author: aapowell
ms.openlocfilehash: 5257d57221e946ac63559b324e7032e1ba78041e
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110069659"
---
# <a name="tutorial-publish-a-hugo-site-to-azure-static-web-apps"></a>Tutorial: Publicación de un sitio de Hugo en Azure Static Web Apps

En este artículo se muestra cómo crear e implementar una aplicación web de [Hugo](https://gohugo.io/) en [Azure Static Web Apps](overview.md). El resultado final es una nueva aplicación de Azure Static Web Apps, con las Acciones de GitHub asociadas, que le da control sobre cómo se compila y publica la aplicación.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> - Crear una aplicación de Hugo
> - Configurar Azure Static Web Apps
> - Implementar la aplicación de Hugo en Azure

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. En caso de no tener una, puede [crear una cuenta gratuita](https://azure.microsoft.com/free/).
- Una cuenta de GitHub. En caso de no tener una, puede [crear una cuenta gratuita](https://github.com/join).

## <a name="create-a-hugo-app"></a>Creación de una aplicación de Hugo

Cree una aplicación de Hugo mediante la interfaz de la línea de comandos (CLI) de Hugo:

1. Siga la [guía de instalación](https://gohugo.io/getting-started/installing/) de Hugo en su sistema operativo.

1. Abra un terminal.

1. Ejecute la CLI de Hugo para crear una aplicación.

   ```bash
   hugo new site static-app
   ```

1. Vaya a la aplicación recién creada.

   ```bash
   cd static-app
   ```

1. Inicialice un repositorio de Git.

   ```bash
    git init
   ```

1. A continuación, agregue un tema al sitio; para ello, instale un tema como submódulo de Git y, luego, especifíquelo en el archivo de configuración de Hugo.

   ```bash
   git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke
   echo 'theme = "ananke"' >> config.toml
   ```

1. Confirme los cambios.

   ```bash
   git add -A
   git commit -m "initial commit"
   ```

## <a name="push-your-application-to-github"></a>Inserción de la aplicación en GitHub

Necesita un repositorio en GitHub para conectarse a Azure Static Web Apps. En los pasos siguientes se muestra cómo crear un repositorio para el sitio.

1. Cree un repositorio de GitHub en blanco (no cree un archivo Léame) desde [https://github.com/new](https://github.com/new), denominado **hugo-static-app**.

1. Agregue el repositorio de GitHub como remoto al repositorio local. Asegúrese de agregar el nombre de usuario de GitHub en lugar del marcador de posición `<YOUR_USER_NAME>` en el comando siguiente.

   ```bash
   git remote add origin https://github.com/<YOUR_USER_NAME>/hugo-static-app
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
    | _Grupos de recursos_ | **my-hugo-group**  |
    | _Nombre_ | **hugo-static-app** |
    | _Tipo de plan_ | **Gratis** |
    | _Región para la API y los entornos de ensayo de Azure Functions_ | Seleccione la región más cercana a la suya. |
    | _Origen_ | **GitHub** |

1. Seleccione **Iniciar sesión con GitHub** y autentíquese con GitHub.

1. Escriba los siguientes valores de GitHub.

    | Propiedad | Valor |
    | --- | --- |
    | _Organización_ | Seleccione la organización de GitHub que quiera. |
    | _Repositorio_ | Seleccione **hugo-static-app**. |
    | _Rama_ | Seleccione **main** (principal). |

1. En la sección _Detalles de la compilación_, seleccione **Hugo** en la lista desplegable _Valores preestablecidos de compilación_ y conserve los valores predeterminados.

### <a name="review-and-create"></a>Revisar y crear

1. Seleccione el botón **Revisar y crear** para comprobar que todos los detalles sean correctos.

1. Seleccione **Crear** para comenzar la creación de la aplicación web estática de App Service y aprovisionar una Acción de GitHub para la implementación.

1. Cuando se complete la implementación, haga clic en **Ir al recurso**.

1. En la pantalla del recurso, haga clic en el vínculo _Dirección URL_ para abrir la aplicación implementada. Es posible que tenga que esperar uno o dos minutos para que se complete la Acción de GitHub.

   :::image type="content" source="./media/publish-hugo/deployed-app.png" alt-text="Aplicación implementada":::

#### <a name="custom-hugo-version"></a>Versión de Hugo personalizada

Cuando se genera una aplicación web estática, se genera un [archivo de flujo de trabajo](./github-actions-workflow.md) que contiene los valores de configuración de publicación de la aplicación. Para designar una versión específica de Hugo en el archivo de flujo de trabajo, proporcione un valor para `HUGO_VERSION` en la sección `env`. En la configuración de ejemplo siguiente se muestra cómo establecer Hugo en una versión específica.

```yaml
jobs:
  build_and_deploy_job:
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
    runs-on: ubuntu-latest
    name: Build and Deploy Job
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true
      - name: Build And Deploy
        id: builddeploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          repo_token: ${{ secrets.GITHUB_TOKEN }} # Used for Github integrations (i.e. PR comments)
          action: "upload"
          ###### Repository/Build Configurations - These values can be configured to match you app requirements. ######
          # For more information regarding Static Web App workflow configurations, please visit: https://aka.ms/swaworkflowconfig
          app_location: "/" # App source code path
          api_location: "api" # Api source code path - optional
          output_location: "public" # Built app content directory - optional
          ###### End of Repository/Build Configurations ######
        env:
          HUGO_VERSION: 0.58.0
```

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cleanup-resource](../../includes/static-web-apps-cleanup-resource.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Agregar un dominio personalizado](custom-domain.md)
