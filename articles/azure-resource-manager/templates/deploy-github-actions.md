---
title: Implementación de plantillas de Resource Manager mediante Acciones de GitHub
description: Se describe cómo implementar plantillas de Azure Resource Manager (plantillas de ARM) mediante Acciones de GitHub.
ms.topic: conceptual
ms.date: 10/13/2020
ms.custom: github-actions-azure, devx-track-azurecli
ms.openlocfilehash: ec29ae019555c54ccdcef9dd743706f8d6401bbd
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107781980"
---
# <a name="deploy-arm-templates-by-using-github-actions"></a>Implementación de plantillas de ARM mediante Acciones de GitHub

[Acciones de GitHub](https://docs.github.com/en/actions) es un conjunto de características de GitHub para automatizar los flujos de trabajo de desarrollo de software en el mismo lugar donde almacena el código y colabora en las solicitudes de incorporación de cambios y problemas.

Use la [acción Implementación de plantilla de Azure Resource Manager](https://github.com/marketplace/actions/deploy-azure-resource-manager-arm-template) para automatizar la implementación de una plantilla de Azure Resource Manager (plantilla de ARM) en Azure.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Una cuenta de GitHub. Si no tiene ninguna, regístrese [gratis](https://github.com/join).
    - Un repositorio de GitHub para almacenar las plantillas de Resource Manager y los archivos del flujo de trabajo. Para crear uno, vea [Creación de un repositorio](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-new-repository).


## <a name="workflow-file-overview"></a>Información general sobre el archivo de flujo de trabajo

Un archivo YAML (.yml) define un flujo de trabajo en la ruta de acceso `/.github/workflows/` de su repositorio. En esta definición se incluyen los diversos pasos y parámetros que componen el flujo de trabajo.

El archivo tiene dos secciones:

|Sección  |Tareas  |
|---------|---------|
|**Autenticación** | 1. Defina una entidad de servicio. <br /> 2. Cree un secreto de GitHub. |
|**Implementar** | 1. Implemente la plantilla de Resource Manager. |

## <a name="generate-deployment-credentials"></a>Genere las credenciales de implementación.


Puede crear una [entidad de servicio](../../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) mediante el comando [az ad sp create-for-rbac](/cli/azure/ad/sp#az_ad_sp_create_for_rbac) de la [CLI de Azure](/cli/azure/). Puede ejecutar este comando mediante [Azure Cloud Shell](https://shell.azure.com/) en Azure Portal o haciendo clic en el botón **Probar**.

Cree un grupo de recursos si todavía no tiene uno.

```azurecli-interactive
    az group create -n {MyResourceGroup} -l {location}
```

Sustituya el marcador de posición `myApp` por el nombre de aplicación.

```azurecli-interactive
   az ad sp create-for-rbac --name {myApp} --role contributor --scopes /subscriptions/{subscription-id}/resourceGroups/{MyResourceGroup} --sdk-auth
```

En el ejemplo anterior, reemplace los marcadores de posición por el identificador de la suscripción y el nombre del grupo de recursos. La salida es un objeto JSON con las credenciales de asignación de roles que proporcionan acceso a la aplicación App Service similar al siguiente. Copie este objeto JSON para más adelante. Solo necesitará las secciones con los valores `clientId`, `clientSecret`, `subscriptionId` y `tenantId`.

```output
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```

> [!IMPORTANT]
> Siempre es recomendable conceder acceso mínimo. El ámbito del ejemplo anterior se limita al grupo de recursos.



## <a name="configure-the-github-secrets"></a>Configuración de los secretos de GitHub

Debe crear secretos para las credenciales de Azure, el grupo de recursos y las suscripciones.

1. En [GitHub](https://github.com/), examine el repositorio.

1. Seleccione **Settings > Secrets > New secret** (Configuración > Secretos > Secreto nuevo).

1. Pegue la salida JSON completa del comando de la CLI de Azure en el campo de valor del secreto. Asigne al secreto el nombre `AZURE_CREDENTIALS`.

1. Cree otro secreto denominado `AZURE_RG`. Agregue el nombre del grupo de recursos al campo de valor del secreto (ejemplo: `myResourceGroup`).

1. Cree un secreto adicional denominado `AZURE_SUBSCRIPTION`. Agregue el identificador de la suscripción al campo de valor del secreto (ejemplo: `90fd3f9d-4c61-432d-99ba-1273f236afa2`).

## <a name="add-resource-manager-template"></a>Adición de la plantilla de Resource Manager

Agregue una plantilla de Resource Manager al repositorio de GitHub. Esta plantilla crea una cuenta de almacenamiento.

```url
https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
```

Puede colocar el archivo en cualquier parte del repositorio. En el ejemplo de flujo de trabajo de la sección siguiente se supone que el archivo de plantilla se denomina **azuredeploy.json** y se almacena en la raíz del repositorio.

## <a name="create-workflow"></a>Creación del flujo de trabajo

El archivo de flujo de trabajo se debe almacenar en la carpeta **.github/workflows** en la raíz del repositorio. La extensión de archivo de flujo de trabajo puede ser **.yml** o **.yaml**.

1. En el repositorio de GitHub, seleccione **Actions** (Acciones) en el menú superior.
1. Seleccione **New workflow** (Nuevo flujo de trabajo).
1. Seleccione **Set up a workflow yourself** (Configurar un flujo de trabajo personalmente).
1. Cambie el nombre del archivo de flujo de trabajo si prefiere otro distinto a **main.yml**. Por ejemplo: **deployStorageAccount.yml**.
1. Reemplace el contenido del archivo .yml por lo siguiente:

    ```yml
    on: [push]
    name: Azure ARM
    jobs:
      build-and-deploy:
        runs-on: ubuntu-latest
        steps:

          # Checkout code
        - uses: actions/checkout@main

          # Log into Azure
        - uses: azure/login@v1
          with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}

          # Deploy ARM template
        - name: Run ARM deploy
          uses: azure/arm-deploy@v1
          with:
            subscriptionId: ${{ secrets.AZURE_SUBSCRIPTION }}
            resourceGroupName: ${{ secrets.AZURE_RG }}
            template: ./azuredeploy.json
            parameters: storageAccountType=Standard_LRS

          # output containerName variable from template
        - run: echo ${{ steps.deploy.outputs.containerName }}
    ```
    > [!NOTE]
    > En su lugar, puede especificar un archivo de parámetros de formato JSON en la acción de implementación de ARM (ejemplo: `.azuredeploy.parameters.json`).

    La primera sección del archivo de flujo de trabajo incluye:

    - **name**: El nombre del flujo de trabajo.
    - **on**: el nombre de los eventos de GitHub que desencadenan el flujo de trabajo. El flujo de trabajo se desencadena cuando hay un evento de envío en la rama principal, que modifica al menos uno de los dos archivos especificados. Los dos archivos son el de trabajo y el de plantilla.

1. Seleccione **Start commit** (Iniciar confirmación).
1. Seleccione **Commit directly to the main branch** (Confirmar directamente en la rama principal).
1. Seleccione **Commit new file** (Confirmar nuevo archivo) (o bien **Commit changes** (Confirmar cambios)).

Como el flujo de trabajo está configurado para que lo desencadene el archivo de flujo de trabajo o el de plantilla que se va a actualizar, el flujo de trabajo se inicia inmediatamente después de confirmar los cambios.

## <a name="check-workflow-status"></a>Comprobación del estado del flujo de trabajo

1. Seleccione la pestaña **Actions** (Acciones). Verá un flujo de trabajo **Create deployStorageAccount.yml** en la lista. El flujo de trabajo tarda un par de minutos en ejecutarse.
1. Seleccione el flujo de trabajo para abrirlo.
1. Seleccione **Run ARM deploy** (Ejecutar implementación de ARM) en el menú para comprobar la implementación.

## <a name="clean-up-resources"></a>Limpieza de recursos
Cuando el repositorio y el grupo de recursos ya no sean necesarios, limpie los recursos que implementó eliminando el grupo de recursos y el repositorio de GitHub.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de la primera plantilla de ARM](./template-tutorial-create-first-template.md)

> [!div class="nextstepaction"]
> [Módulo de aprendizaje: Automatización de la implementación de plantillas de ARM mediante Acciones de GitHub](/learn/modules/deploy-templates-command-line-github-actions/)
