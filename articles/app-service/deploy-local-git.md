---
title: Implementación desde un repositorio GIT local
description: Aprenda a habilitar la implementación de Git local en Azure App Service. Una de las formas más sencillas de implementar código desde una máquina local.
ms.assetid: ac50a623-c4b8-4dfd-96b2-a09420770063
ms.topic: article
ms.date: 02/16/2021
ms.reviewer: dariac
ms.custom: seodec18, devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 3196233728bb7f6493bbc06234c62d261ac99254
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107832400"
---
# <a name="local-git-deployment-to-azure-app-service"></a>Implementación de Git local en Azure App Service

En esta guía se muestra cómo implementar la aplicación en [Azure App Service](overview.md) desde un repositorio Git en la máquina local.

## <a name="prerequisites"></a>Prerrequisitos

Para seguir los pasos de esta guía:

- [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
  
- [Instale Git](https://www.git-scm.com/downloads).

- Obtenga un repositorio Git local con el código que quiere implementar. Para descargar un repositorio de ejemplo, ejecute el comando siguiente en la ventana del terminal local:
  
  ```bash
  git clone https://github.com/Azure-Samples/nodejs-docs-hello-world.git
  ```

[!INCLUDE [Prepare repository](../../includes/app-service-deploy-prepare-repo.md)]

## <a name="configure-a-deployment-user"></a>Configuración de un usuario de implementación

Consulte [Configuración de credenciales de implementación para Azure App Service](deploy-configure-credentials.md). Puede usar credenciales de ámbito de usuario o credenciales de ámbito de aplicación.

## <a name="create-a-git-enabled-app"></a>Creación de una aplicación habilitada para Git

Si ya tiene una aplicación App Service y quiere configurar una implementación de Git local para ella, consulte [Configuración de una aplicación existente](#configure-an-existing-app) en su lugar.

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

Ejecute [`az webapp create`](/cli/azure/webapp#az_webapp_create) con la opción `--deployment-local-git`. Por ejemplo:

```azurecli-interactive
az webapp create --resource-group <group-name> --plan <plan-name> --name <app-name> --runtime "<runtime-flag>" --deployment-local-git
```

La salida contiene una dirección URL, como: `https://<deployment-username>@<app-name>.scm.azurewebsites.net/<app-name>.git`. Use esta dirección URL para implementar la aplicación en el siguiente paso.

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Ejecute [New-AzWebApp](/powershell/module/az.websites/new-azwebapp) desde la raíz del repositorio de Git. Por ejemplo:

```azurepowershell-interactive
New-AzWebApp -Name <app-name>
```

Cuando ejecute este cmdlet desde un directorio que sea un repositorio de Git, se creará automáticamente un repositorio remoto de Git para su aplicación de App Service, denominado `azure`.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

En el portal, primero tiene que crear una aplicación y, a continuación, configurar la implementación para ella. Consulte [Configuración de una aplicación existente](#configure-an-existing-app).

-----

## <a name="configure-an-existing-app"></a>Configuración de una aplicación existente

Si aún no ha creado una aplicación, consulte [Creación de una aplicación habilitada para Git](#create-a-git-enabled-app) en su lugar.

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

Ejecute [`az webapp deployment source config-local-git`](/cli/azure/webapp/deployment/source#az_webapp_deployment_source_config_local_git) . Por ejemplo:

```azurecli-interactive
az webapp deployment source config-local-git --name <app-name> --resource-group <group-name>
```

La salida contiene una dirección URL, como: `https://<deployment-username>@<app-name>.scm.azurewebsites.net/<app-name>.git`. Use esta dirección URL para implementar la aplicación en el siguiente paso.

> [!TIP]
> Esta dirección URL contiene el nombre de usuario de la implementación de ámbito de usuario. Si lo desea, puede [usar las credenciales del ámbito de aplicación](deploy-configure-credentials.md#appscope) en su lugar. 

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Para establecer el `scmType` de la aplicación, ejecute el cmdlet [Set-AzResource](/powershell/module/az.resources/set-azresource).

```powershell-interactive
$PropertiesObject = @{
    scmType = "LocalGit";
}

Set-AzResource -PropertyObject $PropertiesObject -ResourceGroupName <group-name> `
-ResourceType Microsoft.Web/sites/config -ResourceName <app-name>/web `
-ApiVersion 2015-08-01 -Force
```

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

1. En [Azure Portal](https://portal.azure.com), navegue hasta la página de administración de la aplicación.

1. En el menú de la izquierda, seleccione **Centro de implementación** > **Configuración**. Seleccione **Git local** en **Origen** y haga clic en **Guardar**.

    ![Muestra cómo habilitar la implementación de Git local para App Service en Azure Portal](./media/deploy-local-git/enable-portal.png)

1. En la sección de Git local, copie el **URI de Git Clone** para usarlo más adelante. Este URI no contiene ninguna credencial.

-----

## <a name="deploy-the-web-app"></a>Implementación de la aplicación web

1. En una ventana del terminal local, cambie el directorio a la raíz del repositorio de Git y agregue un repositorio remoto de Git mediante la dirección URL que ha recibido de la aplicación. Si el método elegido no le proporciona una dirección URL, use `https://<app-name>.scm.azurewebsites.net/<app-name>.git` con el nombre de la aplicación en `<app-name>`.
   
   ```bash
   git remote add azure <url>
   ```

    > [!NOTE]
    > Si ha [creado una aplicación habilitada para Git en PowerShell mediante New-AzWebApp](#create-a-git-enabled-app), el repositorio remoto ya se ha creado.
   
1. Acceda a la instancia remota de Azure con `git push azure master`. 
   
1. En la ventana **Administrador de credenciales de Git**, escriba las [credenciales de ámbito de usuario o ámbito de aplicación](#configure-a-deployment-user), no las credenciales de inicio de sesión de Azure.

    Si la dirección URL del repositorio remoto de Git ya contiene el nombre de usuario y la contraseña, no se le pedirá. 
   
1. Revise el resultado. Es posible que vea la automatización específica para el entorno de tiempo de ejecución, como MSBuild para ASP.NET, `npm install` para Node.js y `pip install` para Python. 
   
1. Busque su aplicación en Azure Portal para comprobar que el contenido esté implementado.

## <a name="troubleshoot-deployment"></a>Solución de problemas de implementación

Puede ver los siguientes mensajes de error comunes cuando usa Git para publicar en una aplicación de App Service en Azure:

|Message|Causa|Solución
---|---|---|
|`Unable to access '[siteURL]': Failed to connect to [scmAddress]`|La aplicación no está funcionando.|inicie la aplicación en Azure Portal. La implementación de GIT no está disponible cuando la aplicación web está detenida.|
|`Couldn't resolve host 'hostname'`|La información de dirección del repositorio remoto de "azure" es incorrecta.|use el comando `git remote -v` para obtener un listado de todos los remotos, junto con la dirección URL asociada. Compruebe que la URL para el repositorio correcto "azure" es correcta. Si lo necesita, suprima y vuelva a crear este repositorio remoto utilizando la URL correcta.|
|`No refs in common and none specified; doing nothing. Perhaps you should specify a branch such as 'main'.`|No especificó una rama durante `git push`, o no ha establecido el valor `push.default` en `.gitconfig`.|Vuelva a ejecutar `git push` y especifique la rama principal: `git push azure main`.|
|`Error - Changes committed to remote repository but deployment to website failed.`|Insertó una rama local que no coincide con la rama de implementación de la aplicación en "azure".|Compruebe que la rama actual sea `master`. Para cambiar la rama predeterminada, use la configuración de aplicación `DEPLOYMENT_BRANCH`.|
|`src refspec [branchname] does not match any.`|Ha intentado agregar una rama que no es la principal en el repositorio remoto "azure".|Vuelva a ejecutar `git push` y especifique la rama principal: `git push azure main`.|
|`RPC failed; result=22, HTTP code = 5xx.`|este error puede producirse si se trata de insertar un repositorio Git de gran tamaño a través de HTTPS.|Cambie la configuración de Git en la máquina local para aumentar el tamaño de `postBuffer`. Por ejemplo: `git config --global http.postBuffer 524288000`.|
|`Error - Changes committed to remote repository but your web app not updated.`|Implementó una aplicación Node.js con un archivo _package.json_ que especifica los módulos adicionales requeridos.|Revise los mensajes de error `npm ERR!` anteriores a este error para obtener más contexto sobre el error. A continuación se indican las causas conocidas de este error y los mensajes `npm ERR!`:<br /><br />**El archivo package.json tiene una estructura incorrecta**: `npm ERR! Couldn't read dependencies.`<br /><br />**Módulo nativo que no tiene una distribución binaria para Windows**:<br />`npm ERR! \cmd "/c" "node-gyp rebuild"\ failed with 1` <br />or <br />`npm ERR! [modulename@version] preinstall: \make || gmake\ `|

## <a name="additional-resources"></a>Recursos adicionales

- [Servidor de compilación de App Service (documentación de Project Kudu)](https://github.com/projectkudu/kudu/wiki)
- [Implementación continua en Azure App Service](deploy-continuous-deployment.md)
- [Ejemplo: Creación de una aplicación web e implementación de código desde un repositorio local de Git (CLI de Azure)](./scripts/cli-deploy-local-git.md?toc=%2fcli%2fazure%2ftoc.json)
- [Ejemplo: Creación de una aplicación web e implementación de código desde un repositorio local de Git (PowerShell)](./scripts/powershell-deploy-local-git.md?toc=%2fpowershell%2fmodule%2ftoc.json)
