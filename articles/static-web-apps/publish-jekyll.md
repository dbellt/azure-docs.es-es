---
title: 'Tutorial: Publicación de un sitio de Jekyll en Azure Static Web Apps'
description: Aprenda a implementar una aplicación de Jekyll en Azure Static Web Apps.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 04/28/2021
ms.author: cshoe
ms.openlocfilehash: 0f572d49867fe9149416664a405309253dd01af2
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108202951"
---
# <a name="tutorial-publish-a-jekyll-site-to-azure-static-web-apps-preview"></a>Tutorial: Publicación de un sitio de Jekyll en Azure Static Web Apps, versión preliminar

En este artículo se muestra cómo crear e implementar una aplicación web de [Jekyll](https://jekyllrb.com/) en [Azure Static Web Apps](overview.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> - Crear un sitio web de Jekyll.
> - Configurar un recurso de Azure Static Web Apps
> - Implementar la aplicación de Jekyll en Azure.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

- Instalar [Jekyll](https://jekyllrb.com/docs/installation/)
  - Puede usar el subsistema de Windows para Linux y seguir las instrucciones de Ubuntu si fuera necesario.
- Una cuenta de Azure con una suscripción activa. En caso de no tener una, puede [crear una cuenta gratuita](https://azure.microsoft.com/free/).
- Una cuenta de GitHub. En caso de no tener una, puede [crear una cuenta gratuita](https://github.com/join).

## <a name="create-jekyll-app"></a>Creación de una aplicación de Jekyll

Cree una aplicación de Jekyll mediante la interfaz de la línea de comandos (CLI) de Jekyll:

1. En el terminal, ejecute Jekyll CLI para crear una aplicación.

   ```bash
   jekyll new static-app
   ```

1. Vaya a la aplicación recién creada.

   ```bash
   cd static-app
   ```

1. Inicialice un nuevo repositorio de Git.

   ```bash
    git init
   ```

1. Confirme los cambios.

   ```bash
   git add -A
   git commit -m "initial commit"
   ```

## <a name="push-your-application-to-github"></a>Inserción de la aplicación en GitHub

Azure Static Web Apps usa GitHub para publicar su sitio web. En los pasos siguientes se muestra cómo crear un repositorio de GitHub.

1. Cree un repositorio de GitHub en blanco (no cree un archivo Léame) desde [https://github.com/new](https://github.com/new), denominado **jekyll-azure-static**.

1. Agregue el repositorio de GitHub como remoto al repositorio local. Asegúrese de agregar el nombre de usuario de GitHub en lugar del marcador de posición `<YOUR_USER_NAME>` en el comando siguiente.

   ```bash
   git remote add origin https://github.com/<YOUR_USER_NAME>/jekyll-static-app
   ```

1. Inserte el repositorio local en GitHub.

   ```bash
   git push --set-upstream origin main
   ```

   > [!NOTE]
   > Es posible que su rama de git tenga un nombre distinto a `main`. Reemplace `main` en este comando por el valor correcto.

## <a name="deploy-your-web-app"></a>Implementación de la aplicación web

En los pasos siguientes se muestra cómo crear una aplicación de sitio estática e implementarla en un entorno de producción.

### <a name="create-the-application"></a>Creación de la aplicación

1. Acceda a [Azure Portal](https://portal.azure.com).

1. Haga clic en **Crear un recurso**.

1. Busque **Static Web Apps**.

1. Haga clic en **Static Web App (Preview)** (Static Web Apps [versión preliminar]).

1. Haga clic en **Crear**.

1. En **Suscripción**, acepte la suscripción que aparece o seleccione otra en la lista desplegable.

1. En _Grupo de recursos_, seleccione **Nuevo**. En _Nuevo nombre de grupo de recursos_, escriba **jekyll-static-app** y seleccione **Aceptar**.

1. Después, especifique un nombre para la aplicación en el cuadro _Nombre_. Los caracteres válidos incluyen `a-z`, `A-Z`, `0-9` y `-`.

1. En _Región_, seleccione una región cercana disponible.

1. En _SKU_, seleccione **Gratis**.

1. En _Detalles de implementación_, seleccione **GitHub** en _Origen_.

1. Haga clic en el botón **Iniciar sesión con GitHub**.

1. Seleccione la **Organización** en la que creó el repositorio.

1. Seleccione **jekyll-static-app** en _Repositorio_.

1. En _Rama_, seleccione **principal**.

### <a name="build"></a>Build

A continuación, agregue los valores de configuración que el proceso de compilación usa para compilar la aplicación. La configuración siguiente define el archivo de flujo de trabajo de la Acción de GitHub.

1. En _Build Presets_ (Valores preestablecidos de compilación), seleccione **Custom** (Personalizado).

1. Establezca _Ubicación de la aplicación_ en **/**

1. En _Ubicación de salida_, seleccione **_site**.

   No es necesario un valor para _Ubicación de la API_, ya que por el momento no está implementando una API.

   :::image type="content" source="./media/publish-jekyll/github-actions-inputs.png" alt-text="Entradas de Acciones de GitHub":::

### <a name="review-and-create"></a>Revisar y crear

1. Haga clic en el botón **Revisar y crear** para comprobar que todos los detalles sean correctos.

1. Haga clic en **Crear** para comenzar la creación de la aplicación de Azure Static Web Apps y aprovisionar una Acción de GitHub para la implementación.

1. Espere a que se complete la Acción de GitHub.

1. En la ventana _Información general_ de Azure Portal para el recurso de Azure Static Web Apps recién creado, haga clic en el vínculo _Dirección URL_ para abrir la aplicación implementada.

   :::image type="content" source="./media/publish-jekyll/deployed-app.png" alt-text="Aplicación implementada":::

#### <a name="custom-jekyll-settings"></a>Configuración personalizada de Jekyll

Cuando se genera una aplicación web estática, se genera un [archivo de flujo de trabajo](./github-actions-workflow.md) que contiene los valores de configuración de publicación de la aplicación.

Para configurar variables de entorno, como `JEKYLL_ENV`, agregue una sección `env` a la acción de GitHub de Azure Static Web Apps en el flujo de trabajo.

```yaml
- name: Build And Deploy
   id: builddeploy
   uses: Azure/static-web-apps-deploy@v0.0.1-preview
   with:
      azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
      repo_token: ${{ secrets.GITHUB_TOKEN }} # Used for Github integrations (i.e. PR comments)
      action: "upload"
      ###### Repository/Build Configurations - These values can be configured to match you app requirements. ######
      # For more information regarding Static Web App workflow configurations, please visit: https://aka.ms/swaworkflowconfig
      app_location: "/" # App source code path
      api_location: "" # Api source code path - optional
      output_location: "_site_" # Built app content directory - optional
      ###### End of Repository/Build Configurations ######
   env:
      JEKYLL_ENV: production
```

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cleanup-resource](../../includes/static-web-apps-cleanup-resource.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Agregar un dominio personalizado](custom-domain.md)
