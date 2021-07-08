---
title: 'Tutorial: Publicación de un sitio de Gatsby en Azure Static Web Apps'
description: En este tutorial se muestra cómo implementar una aplicación de Gatsby en Azure Static Web Apps.
services: static-web-apps
author: aaronpowell
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 05/10/2021
ms.author: aapowell
ms.custom: devx-track-js
ms.openlocfilehash: 4c6a68b8db40aa07c251cabab28217143105aab1
ms.sourcegitcommit: 0ce834cd348bb8b28a5f7f612c2807084cde8e8f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109814521"
---
# <a name="tutorial-publish-a-gatsby-site-to-azure-static-web-apps"></a>Tutorial: Publicación de un sitio de Gatsby en Azure Static Web Apps

En este artículo se muestra cómo crear e implementar una aplicación web de [Gatsby](https://gatsbyjs.org) en [Azure Static Web Apps](overview.md). El resultado final es un nuevo sitio de Static Web Apps (con las Acciones de GitHub asociadas) que le da control sobre cómo se compila y publica la aplicación.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> - Crear una aplicación de Gatsby
> - Configurar un sitio de Azure Static Web Apps
> - Implementar la aplicación de Gatsby en Azure

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. En caso de no tener una, puede [crear una cuenta gratuita](https://azure.microsoft.com/free/).
- Una cuenta de GitHub. En caso de no tener una, puede [crear una cuenta gratuita](https://github.com/join).
- [Node.js](https://nodejs.org) instalado.

## <a name="create-a-gatsby-app"></a>Creación de una aplicación de Gatsby

Cree una aplicación de Gatsby mediante la interfaz de la línea de comandos (CLI) de Gatsby:

1. Abra un terminal.
1. Use la herramienta [npx](https://www.npmjs.com/package/npx) para crear una aplicación con la CLI de Gatsby. Esta operación puede tardar unos minutos.

   ```bash
   npx gatsby new static-web-app
   ```

1. Vaya a la aplicación recién creada.

   ```bash
   cd static-web-app
   ```

1. Inicialice un repositorio de Git.

   ```bash
   git init
   git add -A
   git commit -m "initial commit"
   ```

## <a name="push-your-application-to-github"></a>Inserción de la aplicación en GitHub

Debe tener un repositorio en GitHub para crear un recurso de Azure Static Web Apps.

1. Cree un repositorio de GitHub en blanco (no cree un archivo Léame) desde [https://github.com/new](https://github.com/new), denominado **gatsby-static-web-app**.

1. A continuación, agregue el repositorio de GitHub que acaba de crear como remoto al repositorio local. Asegúrese de agregar el nombre de usuario de GitHub en lugar del marcador de posición `<YOUR_USER_NAME>` en el comando siguiente.

   ```bash
   git remote add origin https://github.com/<YOUR_USER_NAME>/gatsby-static-web-app
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
    | _Grupos de recursos_ | **my-gatsby-group**  |
    | _Nombre_ | **my-gatsby-app** |
    | _Tipo de plan_ | **Gratis** |
    | _Región para la API y los entornos de ensayo de Azure Functions_ | Seleccione la región más cercana a la suya. |
    | _Origen_ | **GitHub** |

1. Seleccione **Iniciar sesión con GitHub** y autentíquese con GitHub.

1. Escriba los siguientes valores de GitHub.

    | Propiedad | Valor |
    | --- | --- |
    | _Organización_ | Seleccione la organización de GitHub que quiera. |
    | _Repositorio_ | Seleccione **gatsby-static-web-app**. |
    | _Rama_ | Seleccione **main** (principal). |

1. En la sección _Detalles de la compilación_, seleccione **Gatsby** en la lista desplegable _Valores preestablecidos de compilación_ y conserve los valores predeterminados.

### <a name="review-and-create"></a>Revisar y crear

1. Seleccione el botón **Revisar y crear** para comprobar que todos los detalles sean correctos.

1. Seleccione **Crear** para comenzar la creación de la aplicación web estática de App Service y aprovisionar una Acción de GitHub para la implementación.

1. Cuando se complete la implementación, haga clic en **Ir al recurso**.

1. En la pantalla del recurso, haga clic en el vínculo _Dirección URL_ para abrir la aplicación implementada. Es posible que tenga que esperar uno o dos minutos para que se complete la Acción de GitHub.

   :::image type="content" source="./media/publish-gatsby/deployed-app.png" alt-text="Aplicación implementada":::

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cleanup-resource](../../includes/static-web-apps-cleanup-resource.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Agregar un dominio personalizado](custom-domain.md)
