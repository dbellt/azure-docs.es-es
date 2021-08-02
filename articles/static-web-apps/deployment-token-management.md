---
title: Restablecimiento de tokens de implementación en Azure Static Web Apps
description: Restablecimiento de tokens en un sitio de Azure Static Web Apps
services: static-web-apps
author: webmaxru
ms.author: masalnik
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 1/31/2021
ms.openlocfilehash: 8d516a448240e32f58318689d985bc0aad906b9f
ms.sourcegitcommit: 0ce834cd348bb8b28a5f7f612c2807084cde8e8f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109814017"
---
# <a name="reset-deployment-tokens-in-azure-static-web-apps"></a>Restablecimiento de tokens de implementación en Azure Static Web Apps

Cuando se crea un nuevo sitio de Azure Static Web Apps, Azure genera un token que se usa para identificar la aplicación durante la implementación. Durante el aprovisionamiento, este token se almacena como un secreto en el repositorio de GitHub. En este artículo se explica cómo usar y administrar este token.

Normalmente, no es necesario preocuparse por el token de implementación, pero las siguientes son algunas de las razones por las que puede que necesite recuperarlos o restablecerlo.

* **Compromiso de token**: restablezca el token si se expone a una entidad externa.
* **Implementación desde un repositorio de GitHub independiente**: si realizará la implementación manualmente desde un repositorio de GitHub independiente, debe establecer el token de implementación en el nuevo repositorio.

## <a name="prerequisites"></a>Prerrequisitos

- Un repositorio de GitHub existente configurado con Azure Static Web Apps.
- Consulte [Compilación de su primera aplicación estática](getting-started.md) si no tiene una.

## <a name="reset-a-deployment-token"></a>Restablecimiento de un token de implementación

1. Haga clic en el vínculo **Manage deployment token** (administrar token de implementación) en la página _Información general_ de su sitio de Azure Static Web Apps.

    :::image type="content" source="./media/deployment-token-management/manage-deployment-token-button.png" alt-text="Administración del token de implementación":::

1. Haga clic en el botón **Reset token** (restablecer token).

    :::image type="content" source="./media/deployment-token-management/manage-deployment-token.png" alt-text="Restablecimiento del token de implementación":::

1. Después de mostrar un nuevo token en el campo _Token de implementación_, cópielo. Para hacerlo, haga clic en el icono **Copiar al Portapapeles**.


## <a name="update-a-secret-in-the-github-repository"></a>Actualización de un secreto en el repositorio de GitHub

Para mantener la implementación automatizada en ejecución, después de restablecer un token, debe establecer el nuevo valor en el repositorio de GitHub correspondiente.

1. Vaya al repositorio del proyecto en GitHub y haga clic en la pestaña **Configuración**.
1. Haga clic en el elemento de menú **Secretos**. Encontrará un secreto generado durante el aprovisionamiento de aplicaciones web estáticas denominado _AZURE_STATIC_WEB_APPS_API_TOKEN_... en la sección _Secretos del repositorio_.

    :::image type="content" source="./media/deployment-token-management/github-repo-secrets.png" alt-text="Enumeración de los secretos de repositorio":::

    > [!NOTE]
    > Si ha creado el sitio de Azure Static Web Apps en varias ramas de este repositorio, se mostrarán varios secretos _AZURE_STATIC_WEB_APPS_API_TOKEN_... en esta lista. Para seleccionar el correcto, busque el nombre de archivo que aparece en el campo _Editar flujo de trabajo_ de la pestaña _Información general_ del sitio de Static Web Apps.

1. Haga clic en el botón **Actualizar** para abrir el editor.
1. **Pegue el valor** del token de implementación en el campo _Valor_.
1. Haga clic en **Actualizar el secreto**.

    :::image type="content" source="./media/deployment-token-management/github-update-secret.png" alt-text="Actualización del secreto de repositorio":::

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Publicación desde un generador de sitios estáticos](publish-gatsby.md)
