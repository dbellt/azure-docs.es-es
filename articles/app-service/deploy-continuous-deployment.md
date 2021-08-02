---
title: Configuración de la implementación continua
description: Aprenda a habilitar CI/CD para Azure App Service desde GitHub, BitBucket, Azure Repos u otro repositorios. Seleccione la canalización de compilación que se ajuste a sus necesidades.
ms.assetid: 6adb5c84-6cf3-424e-a336-c554f23b4000
ms.topic: article
ms.date: 03/12/2021
ms.reviewer: dariac
ms.custom: seodec18
ms.openlocfilehash: aa058f0de74941a8ac9fe711c7b6de59240680bb
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110536679"
---
# <a name="continuous-deployment-to-azure-app-service"></a>Implementación continua en Azure App Service

[Azure App Service](overview.md) le permite realizar implementaciones continuas desde los repositorios de [GitHub](https://help.github.com/articles/create-a-repo), [BitBucket](https://confluence.atlassian.com/get-started-with-bitbucket/create-a-repository-861178559.html) y [Azure Repos](/azure/devops/repos/git/creatingrepo) al incorporar las últimas actualizaciones.

> [!NOTE]
> La página **Centro de desarrollo (clásico)** de Azure Portal, una versión anterior de la experiencia de implementación, quedó en desuso en marzo de 2021. Este cambio no afecta a ninguna configuración de implementación existente en la aplicación, y puede continuar con la administración de la implementación de aplicaciones en la página **Centro de implementación** del portal.

[!INCLUDE [Prepare repository](../../includes/app-service-deploy-prepare-repo.md)]

## <a name="configure-deployment-source"></a>Configuración del origen de implementación

1. En [Azure Portal](https://portal.azure.com), vaya a la página de administración de la aplicación App Service.

1. En el menú de la izquierda, haga clic en **Centro de implementación** > **Configuración**. 

1. En **Origen**, seleccione una de las opciones de CI/CD.

    ![Muestra cómo elegir el origen de implementación en el Centro de implementación de Azure App Service](media/app-service-continuous-deployment/choose-source.png)

Elija la pestaña que se corresponda con la selección de los pasos.

# <a name="github"></a>[GitHub](#tab/github)

4. [Acciones de GitHub](#how-the-github-actions-build-provider-works) es el proveedor de compilación predeterminado. Para cambiarlo, haga clic en **Cambiar proveedor** > **Servicio de compilación de App Service** (Kudu) > **Aceptar**.

    > [!NOTE]
    > Para usar Azure Pipelines como proveedor de compilación para la aplicación App Service, no lo configure en App Service. En su lugar, configure CI/CD directamente desde Azure Pipelines. La opción **Azure Pipelines** simplemente apunta a la dirección correcta.

1. Si va a realizar la implementación desde GitHub por primera vez, haga clic en **Autorizar** y siga las indicaciones de autorización. Si quiere realizar la implementación desde un repositorio de usuario diferente, haga clic en **Cambiar cuenta**.

1. Una vez que autorice su cuenta de Azure con GitHub, seleccione la **organización**, el **repositorio** y la **rama** para configurar CI/CD. Si no encuentra ningún repositorio ni ninguna organización, es posible que deba habilitar permisos adicionales en GitHub. Para más información, consulte [Administración del acceso a los repositorios de la organización](https://docs.github.com/organizations/managing-access-to-your-organizations-repositories).

1. Cuando Acciones de GitHub es el proveedor de compilación elegido, puede seleccionar el archivo de flujo de trabajo que quiera con las listas desplegables **Pila en tiempo de ejecución** y **Versión**. Azure confirma este archivo de flujo de trabajo en el repositorio de GitHub seleccionado para controlar las tareas de compilación e implementación. Para ver el archivo antes de guardar los cambios, haga clic en **vista previa del archivo**.

    > [!NOTE]
    > App Service detecta la [configuración de la pila de lenguaje](configure-common.md#configure-language-stack-settings) de la aplicación y selecciona la plantilla de flujo de trabajo más adecuada. Si elige otra plantilla, puede implementar una aplicación que no se ejecute correctamente. Para más información, consulte [Cómo funciona el proveedor de compilación de Acciones de GitHub](#how-the-github-actions-build-provider-works).

1. Haga clic en **Save**(Guardar).
   
    Las nuevas confirmaciones del repositorio y la rama seleccionados ahora se implementan continuamente en su aplicación de App Service. Puede realizar el seguimiento de las confirmaciones y las implementaciones en la pestaña **Registros**.

# <a name="bitbucket"></a>[BitBucket](#tab/bitbucket)

La integración de BitBucket usa los servicios de compilación de App Service (Kudu) para la automatización de la compilación.

4. Si va a realizar la implementación desde BitBucket por primera vez, haga clic en **Autorizar** y siga las indicaciones de autorización. Si quiere realizar la implementación desde un repositorio de usuario diferente, haga clic en **Cambiar cuenta**.

1. Para Bitbucket, seleccione el **equipo** , el **repositorio**  y la **rama**  de Bitbucket que quiere implementar continuamente.

1. Haga clic en **Save**(Guardar).
   
    Las nuevas confirmaciones del repositorio y la rama seleccionados ahora se implementan continuamente en su aplicación de App Service. Puede realizar el seguimiento de las confirmaciones y las implementaciones en la pestaña **Registros**.
   
# <a name="local-git"></a>[Git local](#tab/local)

Vea [Implementación de Git local en Azure App Service](deploy-local-git.md).

# <a name="azure-repos"></a>[Azure Repos](#tab/repos)

> [!NOTE]
> Azure Repos como origen de implementación es la compatibilidad con las aplicaciones de Windows.
>

4. El servicio de compilación de App Service (Kudu) es el proveedor de compilación predeterminado.

    > [!NOTE]
    > Para usar Azure Pipelines como proveedor de compilación para la aplicación App Service, no lo configure en App Service. En su lugar, configure CI/CD directamente desde Azure Pipelines. La opción **Azure Pipelines** simplemente apunta a la dirección correcta.

1. Seleccione la **organización de Azure DevOps** , el **proyecto**, el **repositorio** y la **rama** que quiere implementar continuamente. 

    Si su organización de DevOps no figura en la lista, significa que aún no se ha vinculado a su suscripción de Azure. Para más información, consulte [Creación de una conexión de servicio de Azure](/azure/devops/pipelines/library/connect-to-azure).

-----

## <a name="disable-continuous-deployment"></a>Deshabilitación de la implementación continua

1. En [Azure Portal](https://portal.azure.com), vaya a la página de administración de la aplicación App Service.

1. En el menú de la izquierda, haga clic en **Centro de implementación** > **Configuración** > **Desconectar**. 

    ![Muestra cómo desconectar la sincronización de carpetas en la nube con la aplicación de App Service en Azure Portal.](media/app-service-continuous-deployment/disable.png)

1. De manera predeterminada, el archivo de flujo de trabajo de Acciones de GitHub se conserva en el repositorio, pero seguirá desencadenando la implementación en la aplicación. Para eliminarlo del repositorio, seleccione **Eliminar archivo de flujo de trabajo**.

1. Haga clic en **OK**.

[!INCLUDE [What happens to my app during deployment?](../../includes/app-service-deploy-atomicity.md)]

## <a name="how-the-github-actions-build-provider-works"></a>Cómo funciona el proveedor de compilación de Acciones de GitHub

El proveedor de compilación de Acciones de GitHub es una opción para [CI/CD de GitHub](#configure-deployment-source) y hace lo siguiente para configurar CI/CD:

- Deposita un archivo de flujo de trabajo de Acciones de GitHub en el repositorio de GitHub para controlar las tareas de compilación e implementación en App Service.
- Agrega el perfil de publicación de la aplicación como secreto de GitHub. El archivo de flujo de trabajo usa este secreto para autenticarse con App Service.
- Captura la información de los [registros de ejecución del flujo de trabajo](https://docs.github.com/actions/managing-workflow-runs/using-workflow-run-logs) y la muestra en la pestaña **Registros** del **Centro de implementación** de la aplicación.

Puede personalizar el proveedor de compilación de Acciones de GitHub de las siguientes maneras:

- Personalice el archivo de flujo de trabajo después de que se genere en el repositorio de GitHub. Para más información, consulte [Sintaxis de flujo de trabajo para Acciones de GitHub](https://docs.github.com/actions/reference/workflow-syntax-for-github-actions). Solo tiene que asegurarse de que el flujo de trabajo se implementa en App Service con la acción [azure/webapps-deploy](https://github.com/Azure/webapps-deploy).
- Si la rama seleccionada está protegida, todavía puede obtener una vista previa del archivo de flujo de trabajo sin guardar la configuración y, después, agregarla manualmente al repositorio. Este método no proporciona la integración de registros con Azure Portal.
- En lugar de un perfil de publicación, implemente con una [entidad de servicio](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) en Azure Active Directory.

#### <a name="authenticate-with-a-service-principal"></a>Autenticación con una entidad de servicio

Esta configuración opcional reemplaza la autenticación predeterminada por los perfiles de publicación en el archivo de flujo de trabajo generado.

1. Genere una entidad de servicio mediante el comando [az ad sp create-for-rbac](/cli/azure/ad/sp#az_ad_sp_create_for_rbac) de la [CLI de Azure](/cli/azure/). En el ejemplo siguiente, reemplace *\<subscription-id>* , *\<group-name>* y *\<app-name>* con sus propios valores:

    ```azurecli-interactive
    az ad sp create-for-rbac --name "myAppDeployAuth" --role contributor \
                                --scopes /subscriptions/<subscription-id>/resourceGroups/<group-name>/providers/Microsoft.Web/sites/<app-name> \
                                --sdk-auth
    ```
    
    > [!IMPORTANT]
    > Por seguridad, conceda el acceso mínimo necesario a la entidad de servicio. El ámbito del ejemplo anterior se limita a la aplicación específica de App Service y no a todo el grupo de recursos.
    
1. Guarde la salida JSON completa para el paso siguiente, incluido `{}` de nivel superior.

1. En [GitHub](https://github.com/), examine el repositorio y seleccione **Configuración > Secretos > Agregar un nuevo secreto**.

1. Pegue la salida JSON completa del comando de la CLI de Azure en el campo de valor del secreto. Asigne un nombre al secreto, como `AZURE_CREDENTIALS`.

1. En el archivo de flujo de trabajo generado por el **Centro de implementación**, revise el paso de `azure/webapps-deploy` con código, como en el ejemplo siguiente (que se ha modificado desde un archivo de flujo de trabajo de Node.js):

    ```yaml
    - name: Sign in to Azure 
    # Use the GitHub secret you added
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    - name: Deploy to Azure Web App
    # Remove publish-profile
    - uses: azure/webapps-deploy@v2
      with:
        app-name: '<app-name>'
        slot-name: 'production'
        package: .
    - name: Sign out of Azure
      run: |
        az logout
    ```
    
## <a name="deploy-from-other-repositories"></a>Implementación desde otros repositorios

En el caso de las aplicaciones de Windows, puede configurar manualmente la implementación continua desde un repositorio GIT o de Mercurial en la nube que el portal no admite directamente, como [GitLab](https://gitlab.com/). Para ello, elija Git externo en la lista desplegable **Origen**. Para obtener más información, consulte [Configuración de la implementación continua mediante pasos manuales](https://github.com/projectkudu/kudu/wiki/Continuous-deployment#setting-up-continuous-deployment-using-manual-steps).

## <a name="more-resources"></a>Más recursos

* [Implementación desde Azure Pipelines a Azure App Services](/azure/devops/pipelines/apps/cd/deploy-webdeploy-webapps)
* [Investigar los problemas habituales con la implementación continua](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)
* [Uso de Azure PowerShell](/powershell/azure/)
* [Proyecto Kudu](https://github.com/projectkudu/kudu/wiki)
