---
title: 'Inicio rápido: creación de la primera aplicación web estática con Azure Static Web Apps mediante Azure Portal'
description: Aprenda a implementar un sitio estático en Azure Static Web Apps con Azure Portal.
services: static-web-apps
author: craigshoemaker
ms.author: cshoe
ms.date: 05/07/2021
ms.topic: quickstart
ms.service: static-web-apps
ms.custom:
- mode-portal
ms.openlocfilehash: ce4c37091fc56398c9986396d56e2cc66f9b1d8a
ms.sourcegitcommit: 0ce834cd348bb8b28a5f7f612c2807084cde8e8f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109813957"
---
# <a name="quickstart-building-your-first-static-site-in-the-azure-portal"></a>Inicio rápido: creación del primer sitio estático en Azure Portal

Azure Static Web Apps publica sitios web en entornos de producción mediante la creación de aplicaciones desde un repositorio de GitHub. En este inicio rápido, se implementa una aplicación web en Azure Static Web Apps mediante Azure Portal.

Si no tiene ninguna suscripción a Azure, [cree una cuenta de evaluación gratuita](https://azure.microsoft.com/free).

## <a name="prerequisites"></a>Requisitos previos

- [GitHub](https://github.com)
- Cuenta de [Azure](https://portal.azure.com)

[!INCLUDE [create repository from template](../../includes/static-web-apps-get-started-create-repo.md)]

## <a name="create-a-static-web-app"></a>Creación de una aplicación web estática

Ahora que se ha creado el repositorio, puede crear una aplicación web estática desde Azure Portal.

1. Acceda a [Azure Portal](https://portal.azure.com).
1. Seleccione **Crear un recurso**.
1. Busque **Static Web Apps**.
1. Seleccione **Static Web Apps**.
1. Seleccione **Crear**.

En la sección _Aspectos básicos_, configure la nueva aplicación y vincúlela a un repositorio de GitHub.

:::image type="content" source="media/getting-started-portal/quickstart-portal-basics.png" alt-text="Sección Aspectos básicos":::

1. Seleccione la _suscripción de Azure_.
1. Junto a _Grupo de recursos_, seleccione el vínculo **Crear nuevo**.
1. Escriba **static-web-apps-test** en el cuadro de texto.
1. En _Detalles de la aplicación web estática_, escriba **my-first-static-web-app** en el cuadro de texto.
1. En _Azure Functions and staging details_ (Azure Functions y detalles de almacenamiento provisional), seleccione la región más cercana.
1. En _Detalles de implementación_, seleccione **GitHub**.
1. Seleccione el botón **Sign-in with GitHub** (Iniciar sesión con GitHub) y autentíquese con GitHub.

Tras iniciar sesión con GitHub, escriba la información del repositorio.

:::image type="content" source="media/getting-started-portal/quickstart-portal-source-control.png" alt-text="Detalles del repositorio":::

1. Seleccione el nombre de _Organización_ preferida.
1. Seleccione **my-first-web-static-app** en el menú desplegable _Repositorio_.
1. Seleccione **main** en el menú desplegable _Rama_.

   > [!NOTE]
   > Si no ve ningún repositorio, es posible que deba autorizar Azure Static Web Apps en GitHub. Vaya al repositorio de GitHub y, a continuación, a **Configuración > Aplicaciones > Aplicaciones de OAuth autorizadas**, seleccione **Azure Static Web Apps** y, después, **Conceder**. En el caso de los repositorios de la organización, debe ser propietario de la organización para conceder los permisos.

1. En la sección _Build Details_ (Detalles de la compilación), agregue los detalles de configuración específicos del marco de front-end que prefiera.

    # <a name="no-framework"></a>[Ningún marco](#tab/vanilla-javascript)

    1. Seleccione **Personalizado** en la lista desplegable _Valores preestablecidos de compilación_.
    1. Conserve el valor predeterminado en el cuadro _Ubicación de la aplicación_.
    1. Deje en blanco el cuadro _Ubicación de la aplicación_.
    1. Deje en blanco el cuadro _Ubicación del artefacto de la aplicación_.

    # <a name="angular"></a>[Angular](#tab/angular)

    1. Seleccione **Angular** en la lista desplegable _Valores preestablecidos de compilación_.
    1. Conserve el valor predeterminado en el cuadro _Ubicación de la aplicación_.
    1. Deje en blanco el cuadro _Ubicación de la aplicación_.
    1. Escriba **dist/angular-basic** en el cuadro _Ubicación del artefacto de la aplicación_.

    # <a name="react"></a>[React](#tab/react)

    1. Seleccione **React** en la lista desplegable _Valores preestablecidos de compilación_.
    1. Conserve el valor predeterminado en el cuadro _Ubicación de la aplicación_.
    1. Deje en blanco el cuadro _Ubicación de la aplicación_.
    1. Escriba **build** en el cuadro _Ubicación del artefacto de la aplicación_.

    # <a name="vue"></a>[Vue](#tab/vue)

    1. Seleccione **Vue.js** en la lista desplegable _Valores preestablecidos de compilación_.
    1. Conserve el valor predeterminado en el cuadro _Ubicación de la aplicación_.
    1. Deje en blanco el cuadro _Ubicación de la aplicación_.
    1. Deje el valor predeterminado en el cuadro _Ubicación del artefacto de la aplicación_.

    ---

1. Seleccione **Revisar + crear**.

    :::image type="content" source="media/getting-started-portal/review-create.png" alt-text="Botón Revisar y crear":::

    > [!NOTE]
    > Puede editar el [archivo de flujo de trabajo](github-actions-workflow.md) para cambiar estos valores después de crear la aplicación.

1. Seleccione **Crear**.

    :::image type="content" source="media/getting-started-portal/create-button.png" alt-text="Botón Crear":::

1. Seleccione **Ir al recurso**.

    :::image type="content" source="media/getting-started-portal/resource-button.png" alt-text="Botón Ir al recurso":::

[!INCLUDE [view website](../../includes/static-web-apps-get-started-view-website.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, puede eliminar la instancia de Azure Static Web Apps mediante los siguientes pasos:

1. Abra [Azure Portal](https://portal.azure.com).
1. Busque **my-first-web-static-app** en la barra de búsqueda superior.
1. Seleccione el nombre de la aplicación.
1. Seleccione el botón **Eliminar**.
1. Seleccione **Sí** para confirmar la acción de eliminación (esta acción puede tardar unos minutos en completarse).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Adición de una API](add-api.md)
