---
title: Configuración de las credenciales de implementación
description: Obtenga información acerca de los tipos de credenciales de implementación que se encuentran en Azure App Service y cómo configurarlas y usarlas.
ms.topic: article
ms.date: 02/11/2021
ms.reviewer: byvinyal
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: b77a26f61e1168846156de990806bbed2f7c41e3
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107789544"
---
# <a name="configure-deployment-credentials-for-azure-app-service"></a>Configuración de credenciales de implementación para Azure App Service
Para proteger la implementación de aplicaciones desde un equipo local, [Azure App Service](./overview.md) admite dos tipos de credenciales para la [implementación de Git local](deploy-local-git.md) y la [implementación de FTP/S](deploy-ftp.md). Estas credenciales no son las mismas que las de su suscripción a Azure.

[!INCLUDE [app-service-deploy-credentials](../../includes/app-service-deploy-credentials.md)]

> [!NOTE]
> La página **Centro de desarrollo (clásico)** en Azure Portal, que es la experiencia de implementación anterior, quedará en desuso en marzo de 2021. Este cambio no afectará a ninguna configuración de implementación existente en la aplicación y puede continuar con la administración de la implementación de aplicaciones en la página **Centro de implementación**.

## <a name="configure-user-scope-credentials"></a><a name="userscope"></a>Configuración de credenciales de ámbito de usuario

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

Ejecute el comando [az webapp deployment user set](/cli/azure/webapp/deployment/user#az_webapp_deployment_user_set). Reemplace \<username> y \<password> por un nombre de usuario y una contraseña de usuario de implementación. 

- El nombre de usuario debe ser único dentro de Azure y no debe contener el símbolo "\@" para las inserciones de Git local. 
- La contraseña debe tener al menos ocho caracteres y dos de los tres elementos siguientes: letras, números y símbolos. 

```azurecli-interactive
az webapp deployment user set --user-name <username> --password <password>
```

La salida JSON muestra la contraseña como `null`.

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

No se pueden configurar las credenciales de ámbito de usuario con Azure PowerShell. Use otro método o considere la posibilidad de [usar credenciales de ámbito de aplicación](#appscope). 

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Puede configurar las credenciales de ámbito de usuario en cualquier [página de recursos](../azure-resource-manager/management/manage-resources-portal.md#manage-resources) de la aplicación. Independientemente de en qué aplicación configure estas credenciales, son válidas para todas las aplicaciones de todas las suscripciones de su cuenta de Azure. 

En [Azure Portal](https://portal.azure.com), debe tener al menos una aplicación para poder acceder a la página de credenciales de implementación. Para configurar las credenciales de ámbito de usuario:

1. En el menú de la izquierda de la aplicación, seleccione **Centro de implementación** > **Credenciales de FTPS** o **Credenciales GIT o FTPS locales**.

    ![Muestra cómo puede seleccionar el panel FTP desde el centro de implementación en Azure App Services.](./media/app-service-deployment-credentials/access-no-git.png)

2. Desplácese hacia abajo hasta **Ámbito de usuario**, configure el **nombre de usuario** y la **contraseña** y, a continuación, seleccione **Guardar**.

Una vez que haya configurado las credenciales de implementación, puede encontrar el nombre de usuario de implementación de *Git* en la página **Información general** de la aplicación.

![Muestra cómo buscar el nombre de usuario de implementación de GIT en la página de información general de la aplicación.](./media/app-service-deployment-credentials/deployment_credentials_overview.png)

Si se configura la implementación de Git, la página muestra un **nombre de usuario de implementación o Git**; de lo contrario, un **nombre de usuario de implementación o FTP**.

> [!NOTE]
> Azure no muestra la contraseña de la implementación de ámbito de usuario. Si se le olvida la contraseña, podrá restablecer las credenciales siguiendo los pasos descritos en esta sección.
>
> 

-----

## <a name="use-user-scope-credentials-with-ftpftps"></a>Uso de credenciales de ámbito de usuario con FTP/FTPS

La autenticación en un punto de conexión FTP o FTPS con credenciales de ámbito de usuario requiere un nombre de usuario con el formato siguiente: `<app-name>\<user-name>`

Puesto que las credenciales de ámbito de usuario están vinculadas al usuario y no a un recurso específico, el nombre de usuario debe tener este formato para dirigir la acción de inicio de sesión al punto de conexión de la aplicación adecuada.

## <a name="get-application-scope-credentials"></a><a name="appscope"></a>Obtención de credenciales de ámbito de aplicación

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

Obtenga las credenciales del ámbito de aplicación mediante el comando [az webapp deployment list-publishing-profiles](/cli/azure/webapp/deployment#az_webapp_deployment_list_publishing_profiles). Por ejemplo:

```azurecli-interactive
az webapp deployment list-publishing-profiles --resource-group <group-name> --name <app-name>
```

Para la [implementación de Git local](deploy-local-git.md), también puede usar el comando [az webapp deployment list-publishing-credentials](/cli/azure/webapp/deployment#az_webapp_deployment_list_publishing_credentials) para obtener un URI de repositorio remoto de Git para la aplicación, con las credenciales del ámbito de aplicación ya insertadas. Por ejemplo:

```azurecli-interactive
az webapp deployment list-publishing-credentials --resource-group <group-name> --name <app-name> --query scmUri
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Obtenga las credenciales del ámbito de aplicación mediante el comando [Get-AzWebAppPublishingProfile](/powershell/module/az.websites/get-azwebapppublishingprofile). Por ejemplo:

```azurepowershell-interactive
Get-AzWebAppPublishingProfile -ResourceGroupName <group-name> -Name <app-name>
```

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

1. En el menú de la izquierda de la aplicación, seleccione **Centro de implementación** > **Credenciales de FTPS** o **Credenciales GIT o FTPS locales**.

    ![Muestra cómo puede seleccionar el panel FTP desde el centro de implementación en Azure App Services.](./media/app-service-deployment-credentials/access-no-git.png)

2. En la sección **Ámbito de aplicación**, seleccione el vínculo **Copiar** para copiar el nombre de usuario o la contraseña.

-----

## <a name="reset-application-scope-credentials"></a>Restablecimiento de credenciales de ámbito de aplicación

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

Restablezca las credenciales del ámbito de aplicación mediante el comando [az resource invoke-action](/cli/azure/resource#az_resource_invoke_action):

```azurecli-interactive
az resource invoke-action --action newpassword --resource-group <group-name> --name <app-name> --resource-type Microsoft.Web/sites
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Restablezca las credenciales del ámbito de aplicación mediante el comando [Invoke-AzResourceAction](/powershell/module/az.resources/invoke-azresourceaction):

```azurepowershell-interactive
Invoke-AzResourceAction -ResourceGroupName <group-name> -ResourceType Microsoft.Web/sites -ResourceName <app-name> -Action newpassword
```

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

1. En el menú de la izquierda de la aplicación, seleccione **Centro de implementación** > **Credenciales de FTPS** o **Credenciales GIT o FTPS locales**.

    ![Muestra cómo puede seleccionar el panel FTP desde el centro de implementación en Azure App Services.](./media/app-service-deployment-credentials/access-no-git.png)

2. En la sección **Ámbito de aplicación**, seleccione **Restablecer**.

-----

## <a name="disable-basic-authentication"></a>Deshabilitación de la autenticación básica

Algunas organizaciones deben cumplir los requisitos de seguridad y preferirán deshabilitar el acceso a través de FTP o WebDeploy. De este modo, los miembros de la organización solo pueden tener acceso a su instancia de App Services a través de las API que están controladas por Azure Active Directory (Azure AD).

### <a name="ftp"></a>FTP

Para deshabilitar el acceso FTP al sitio, ejecute el siguiente comando de la CLI. Reemplace los marcadores de posición por el grupo de recursos y el nombre del sitio. 

```azurecli-interactive
az resource update --resource-group <resource-group> --name ftp --namespace Microsoft.Web --resource-type basicPublishingCredentialsPolicies --parent sites/<site-name> --set properties.allow=false
```

Para confirmar que el acceso FTP está bloqueado, puede intentar realizar la autenticación mediante un cliente FTP como FileZilla. Para recuperar las credenciales de publicación, vaya a la hoja de información general de su sitio y haga clic en Descargar perfil de publicación. Use el nombre de host, el nombre de usuario y la contraseña de FTP del archivo para autenticarse; recibirá una respuesta de error 401 que indica que no está autorizado.

### <a name="webdeploy-and-scm"></a>WebDeploy y SCM

Para deshabilitar el acceso de autenticación básica al puerto de WebDeploy y al sitio de SCM, ejecute el siguiente comando de la CLI. Reemplace los marcadores de posición por el grupo de recursos y el nombre del sitio. 

```azurecli-interactive
az resource update --resource-group <resource-group> --name scm --namespace Microsoft.Web --resource-type basicPublishingCredentialsPolicies --parent sites/<site-name> --set properties.allow=false
```

Para confirmar que las credenciales del perfil de publicación están bloqueadas en WebDeploy, pruebe a [publicar una aplicación web con Visual Studio 2019](/visualstudio/deployment/quickstart-deploy-to-azure).

### <a name="disable-access-to-the-api"></a>Deshabilitación del acceso a la API

La API de la sección anterior está respaldada por el control de acceso basado en rol (RBAC) de Azure, lo que significa que puede [crear un rol personalizado](../role-based-access-control/custom-roles.md#steps-to-create-a-custom-role) y asignarlo a usuarios con pocos privilegios para que no puedan habilitar la autenticación básica en ningún sitio. Para configurar el rol personalizado, [siga estas instrucciones](https://azure.github.io/AppService/2020/08/10/securing-data-plane-access.html#create-a-custom-rbac-role).

También puede usar [Azure Monitor](https://azure.github.io/AppService/2020/08/10/securing-data-plane-access.html#audit-with-azure-monitor) para auditar las solicitudes de autenticación correctas y [Azure Policy](https://azure.github.io/AppService/2020/08/10/securing-data-plane-access.html#enforce-compliance-with-azure-policy) para aplicar esta configuración a todos los sitios de la suscripción.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo usar estas credenciales para implementar la aplicación desde [GIT local](deploy-local-git.md) o con [FTP/S](deploy-ftp.md).
