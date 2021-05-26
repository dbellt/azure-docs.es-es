---
title: 'Tutorial: Publicación de aplicaciones web estáticas de Azure con Azure DevOps'
description: Aprenda a usar Azure DevOps para publicar aplicaciones web estáticas de Azure.
services: static-web-apps
author: scubaninja
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 03/23/2021
ms.author: apedward
ms.openlocfilehash: 393b8857b3602d914143787cc9ea46074ff59c05
ms.sourcegitcommit: 0ce834cd348bb8b28a5f7f612c2807084cde8e8f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109813909"
---
# <a name="tutorial-publish-azure-static-web-apps-with-azure-devops"></a>Tutorial: Publicación de aplicaciones web estáticas de Azure con Azure DevOps

En este artículo se muestra cómo implementar [aplicaciones web estáticas de Azure](./overview.md) mediante [Azure DevOps](https://dev.azure.com/).

En este tutorial, aprenderá a:

- Configurar un sitio de Azure Static Web Apps
- Crear una canalización de Azure DevOps para crear y publicar una aplicación web estática

## <a name="prerequisites"></a>Prerrequisitos

- Una **cuenta activa de Azure**: en caso de no tener una, puede [crear una cuenta gratuita](https://azure.microsoft.com/free/).
- Un **proyecto de Azure DevOps:** si no tiene uno, puede [crear un proyecto de forma gratuita](https://azure.microsoft.com/pricing/details/devops/azure-devops-services/).
- Una **canalización de Azure DevOps:** si necesita ayuda para comenzar, consulte [Creación de la primera canalización](/azure/devops/pipelines/create-first-pipeline?preserve-view=true&view=azure-devops).

## <a name="create-a-static-web-app-in-an-azure-devops-repository"></a>Creación de una aplicación web estática en un repositorio de Azure DevOps

  > [!NOTE]
  > Si tiene una aplicación existente en el repositorio, puede ir directamente a la sección siguiente.

1. Vaya al repositorio de Azure DevOps.

1. Seleccione **Importar** para iniciar la importación de una aplicación de ejemplo.
  
    :::image type="content" source="media/publish-devops/devops-repo.png" alt-text="Repositorio de DevOps":::

1. En **Dirección URL de clonación**, escriba `https://github.com/staticwebdev/vanilla-api.git`.

1. Seleccione **Import** (Importar).

## <a name="create-a-static-web-app"></a>Creación de una aplicación web estática

1. Acceda a [Azure Portal](https://portal.azure.com).

1. Seleccione **Crear un recurso**.

1. Busque **Static Web Apps**.

1. Seleccione **Static Web Apps**.

1. Seleccione **Crear**.

1. En _Detalles de la implementación_, asegúrese de que selecciona **Otro**. Esto le permite usar el código en el repositorio de Azure DevOps.

    :::image type="content" source="media/publish-devops/create-resource.png" alt-text="Detalles de la implementación - Otro":::

1. Una vez que la implementación se haya realizado correctamente, vaya al nuevo recurso de aplicaciones web estáticas.

1. Seleccione **Manage deployment token** (Administrar token de implementación).

1. Copie el **token de implementación** y péguelo en un editor de texto para usarlo en otra pantalla.

    > [!NOTE]
    > Este valor se reserva ahora porque copiará y pegará más valores en los pasos siguientes.

    :::image type="content" source="media/publish-devops/deployment-token.png" alt-text="Token de implementación":::

## <a name="create-the-pipeline-task-in-azure-devops"></a>Creación de la tarea de canalización de Azure DevOps

1. Vaya al repositorio de Azure DevOps que se creó anteriormente.

1. Seleccione **Configurar la compilación**.

    :::image type="content" source="media/publish-devops/azdo-build.png" alt-text="Canalización de compilación":::

1. En la pantalla *Configurar la canalización*, seleccione **Canalización inicial**.

    :::image type="content" source="media/publish-devops/configure-pipeline.png" alt-text="Configurar canalización":::

1. Copie y pegue el siguiente YAML en la canalización.

    ```yaml
    trigger:
    - main

    pool:
      vmImage: ubuntu-latest

    steps:
      - checkout: self
        submodules: true
      - task: AzureStaticWebApp@0
        inputs:
          app_location: '/'
          api_location: 'api'
          output_location: ''
        env:
          azure_static_web_apps_api_token: $(deployment_token)
    ```

    > [!NOTE]
    > Si no va a usar la aplicación de ejemplo, los valores de `app_location`, `api_location` y `output_location` deben cambiar para que coincidan con los valores de la aplicación.

    [!INCLUDE [static-web-apps-folder-structure](../../includes/static-web-apps-folder-structure.md)]

    El valor `azure_static_web_apps_api_token` es autoadministrado y se configura manualmente.

2. Seleccione **Variables**.

3. Cree una nueva variable.

4. Asigne a la variable el nombre **deployment_token** (que coincide con el nombre del flujo de trabajo).

5. Copie el token de implementación que pegó anteriormente en un editor de texto.

6. Pegue el token de implementación en el cuadro _Valor_.

    :::image type="content" source="media/publish-devops/variable-token.png" alt-text="Token de variable":::

7. Seleccione **Mantener este valor como secreto**.

8. Seleccione **Aceptar**.

9. Seleccione **Guardar** para volver al archivo YAML de la canalización.

10. Seleccione **Guardar y ejecutar** para que se abra el cuadro de diálogo del mismo nombre.

    :::image type="content" source="media/publish-devops/save-and-run.png" alt-text="Canalización":::

11. Seleccione **Guardar y ejecutar** para ejecutar la canalización.

12. Una vez que la implementación se haya realizado correctamente, vaya a la página de **información general** de Azure Static Web Apps, que incluye vínculos a la configuración de implementación. Observe cómo el vínculo de _origen_ apunta ahora a la rama y ubicación del repositorio de Azure DevOps.

13. Seleccione la **dirección URL** para ver el sitio web recién implementado.

    :::image type="content" source="media/publish-devops/deployment-location.png" alt-text="Ubicación de implementación":::

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración de Azure Static Web Apps](./configuration.md)