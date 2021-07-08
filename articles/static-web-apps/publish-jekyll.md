---
title: 'Tutorial: Publicación de un sitio de Jekyll en Azure Static Web Apps'
description: Aprenda a implementar una aplicación de Jekyll en Azure Static Web Apps.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 05/11/2021
ms.author: cshoe
ms.openlocfilehash: 179fa0e247b2c875a4d32eac312d240ae768c009
ms.sourcegitcommit: a9f131fb59ac8dc2f7b5774de7aae9279d960d74
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110190551"
---
# <a name="tutorial-publish-a-jekyll-site-to-azure-static-web-apps"></a>Tutorial: Publicación de un sitio de Jekyll en Azure Static Web Apps

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

1. Vaya a [Azure Portal](https://portal.azure.com).
1. Seleccione **Crear un recurso**
1. Busque **Static Web Apps**.
1. Seleccione **Static Web Apps**.
1. Seleccione **Crear**
1. En la pestaña _Datos básicos_, especifique los valores siguientes.

    | Propiedad | Valor |
    | --- | --- |
    | _Suscripción_ | El nombre de la suscripción de Azure. |
    | _Grupos de recursos_ | **jekyll-static-app**  |
    | _Nombre_ | **jekyll-static-app** |
    | _Tipo de plan_ | **Gratis** |
    | _Región para la API y los entornos de ensayo de Azure Functions_ | Seleccione la región más cercana a la suya. |
    | _Origen_ | **GitHub** |

1. Seleccione **Iniciar sesión con GitHub** y autentíquese con GitHub.

1. Escriba los siguientes valores de GitHub.

    | Propiedad | Valor |
    | --- | --- |
    | _Organización_ | Seleccione la organización de GitHub que quiera. |
    | _Repositorio_ | Seleccione **jekyll-static-app**. |
    | _Rama_ | Seleccione **main** (principal). |

1. En la sección _Detalles de la compilación_, seleccione **Personalizado** en la lista desplegable _Valores preestablecidos de compilación_ y conserve los valores predeterminados.

1. En el cuadro _Ubicación de la aplicación_, escriba **./** .

1. Deje en blanco el cuadro _Ubicación de la aplicación_.

1. En el cuadro _Ubicación del resultado_, escriba **_site**.

### <a name="review-and-create"></a>Revisar y crear

1. Seleccione el botón **Revisar y crear** para comprobar que todos los detalles sean correctos.

1. Seleccione **Crear** para comenzar la creación de la aplicación web estática de App Service y aprovisionar una Acción de GitHub para la implementación.

1. Cuando se complete la implementación, haga clic en **Ir al recurso**.

1. En la pantalla del recurso, haga clic en el vínculo _Dirección URL_ para abrir la aplicación implementada. Es posible que tenga que esperar uno o dos minutos para que se complete la Acción de GitHub.

   :::image type="content" source="./media/publish-jekyll/deployed-app.png" alt-text="Aplicación implementada":::

#### <a name="custom-jekyll-settings"></a>Configuración personalizada de Jekyll

Cuando se genera una aplicación web estática, se genera un [archivo de flujo de trabajo](./github-actions-workflow.md) que contiene los valores de configuración de publicación de la aplicación.

Para configurar variables de entorno, como `JEKYLL_ENV`, agregue una sección `env` a la acción de GitHub de Azure Static Web Apps en el flujo de trabajo.

```yaml
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
      api_location: "" # Api source code path - optional
      output_location: "_site" # Built app content directory - optional
      ###### End of Repository/Build Configurations ######
   env:
      JEKYLL_ENV: production
```

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cleanup-resource](../../includes/static-web-apps-cleanup-resource.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Agregar un dominio personalizado](custom-domain.md)
