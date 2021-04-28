---
title: Implementación de contenido mediante FTP/S
description: Aprenda a implementar la aplicación en Azure App Service mediante FTP o FTPS. Mejora de la seguridad del sitio web mediante la deshabilitación del FTP sin cifrar.
ms.assetid: ae78b410-1bc0-4d72-8fc4-ac69801247ae
ms.topic: article
ms.date: 02/26/2021
ms.reviewer: dariac
ms.custom: seodec18, devx-track-azurepowershell
ms.openlocfilehash: fd856ee47c1100292f7558bb0fa2a1772951a3cb
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107832376"
---
# <a name="deploy-your-app-to-azure-app-service-using-ftps"></a>Implementación de la aplicación en Azure App Service mediante FTP/S

En este artículo se muestra cómo usar FTP o FTPS para implementar la aplicación web, el back-end de la aplicación móvil o la aplicación de API en [Azure App Service](./overview.md).

El punto de conexión FTP/S de la aplicación ya está activo. No se necesita ninguna configuración para habilitar la implementación de FTP/S.

> [!NOTE]
> La página **Centro de desarrollo (clásico)** en Azure Portal, que es la experiencia de implementación anterior, quedará en desuso en marzo de 2021. Este cambio no afectará a ninguna configuración de implementación existente en la aplicación y puede continuar con la administración de la implementación de aplicaciones en la página **Centro de implementación**.

## <a name="get-deployment-credentials"></a>Obtención de credenciales de implementación

1. Siga las instrucciones de [Configuración de credenciales de implementación para Azure App Service](deploy-configure-credentials.md) para copiar las credenciales del ámbito de aplicación o establecer las credenciales del ámbito de usuario. Puede conectarse al punto de conexión FTP/S de la aplicación mediante cualquiera de esas credenciales.

1. Cree el nombre de usuario de FTP con el siguiente formato, según su elección del ámbito de las credenciales:

    | Ámbito de aplicación | Ámbito de usuario |
    | - | - |
    |`<app-name>\$<app-name>`|`<app-name>\<deployment-user>`|

    ---

    En App Service, el punto de conexión FTP/S se comparte entre las aplicaciones. Dado que las credenciales de ámbito de usuario no están vinculadas a un recurso específico, el nombre de la aplicación se debe anteponer al nombre de usuario del ámbito de usuario, tal como se mostró anteriormente.

## <a name="get-ftps-endpoint"></a>Obtención del punto de conexión FTP/S
    
# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

En la misma página de administración de la aplicación en la que ha copiado las credenciales de implementación [**Centro de implementación** > **FTP Credentials** (Credenciales de FTP)], copie el **Punto de conexión de FTPS**.

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

Ejecute el comando [az webapp deployment list-publishing-profiles](/cli/azure/webapp/deployment#az_webapp_deployment_list_publishing_profiles). En el ejemplo siguiente se usa una [ruta de acceso JMES](https://jmespath.org/) para extraer los puntos de conexión de FTP/S de la salida.

```azurecli-interactive
az webapp deployment list-publishing-profiles --name <app-name> --resource-group <group-name> --query "[?ends_with(profileName, 'FTP')].{profileName: profileName, publishUrl: publishUrl}"
```

Cada aplicación tiene dos puntos de conexión de FTP/S, uno es de lectura y escritura, mientras que el otro es de solo lectura (`profileName` contiene `ReadOnly`) y es para escenarios de recuperación de datos. Para implementar archivos con FTP, copie la dirección URL del punto de conexión de lectura y escritura.

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Ejecute el comando [Get-AzWebAppPublishingProfile](/powershell/module/az.websites/get-azwebapppublishingprofile). En el ejemplo siguiente se extrae el punto de conexión FTP/S de la salida XML.

```azurepowershell-interactive
$xml = [xml](Get-AzWebAppPublishingProfile -Name <app-name> -ResourceGroupName <group-name> -OutputFile null)
$xml.SelectNodes("//publishProfile[@publishMethod=`"FTP`"]/@publishUrl").value
```

-----

## <a name="deploy-files-to-azure"></a>Implementación de archivos en Azure

1. Desde el cliente de FTP (por ejemplo, [Visual Studio](https://www.visualstudio.com/vs/community/), [Cyberduck](https://cyberduck.io/) o [WinSCP](https://winscp.net/index.php)), use la información de conexión recopilada para conectarse a la aplicación.
2. Copie los archivos y la estructura de directorio correspondiente al directorio [ **/site/wwwroot**](https://github.com/projectkudu/kudu/wiki/File-structure-on-azure)en Azure (o el directorio **/site/wwwroot/App_Data/Jobs/** para WebJobs).
3. Vaya a la dirección URL de la aplicación para comprobar que la aplicación se está ejecutando correctamente. 

> [!NOTE] 
> A diferencia de las [implementaciones basadas en Git](deploy-local-git.md) y de la [implementación de ZIP](deploy-zip.md), la implementación de FTP no admite la automatización de la compilación, como: 
>
> - restauraciones de dependencias (por ejemplo, automatizaciones de NuGet, NPM, PIP y Composer)
> - compilación de archivos binarios de .NET
> - generación del archivo web.config (aquí se muestra un [ejemplo de Node.js](https://github.com/projectkudu/kudu/wiki/Using-a-custom-web.config-for-Node-apps))
> 
> Genere estos archivos necesarios de forma manual en la máquina local e impleméntelos luego junto con la aplicación.
>

## <a name="enforce-ftps"></a>Aplicación de FTPS

Para mejorar la seguridad, permita FTP a través de TLS/SSL únicamente. También puede deshabilitar FTP y FTPS si no utiliza la implementación de FTP.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

1. En la página de recursos de la aplicación de [Azure Portal](https://portal.azure.com), seleccione **Configuración** > **Configuración general** en el panel de navegación izquierdo.

2. Para deshabilitar FTP sin cifrar, seleccione **Solo FTPS** en **Estado FTP**. Para deshabilitar completamente tanto FTP como FTPS, seleccione **Deshabilitado**. Cuando termine, haga clic en **Guardar**. Si usa **Solo FTPS** debe exigir TLS 1.2, o cualquier versión posterior; para ello, navegue a la hoja **Configuración de TLS/SSL** de la aplicación web. TLS 1.0 y 1.1 no son compatibles con **Solo FTPS**.

    ![Deshabilitación de FTP/S](./media/app-service-deploy-ftp/disable-ftp.png)

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

Ejecute el comando [az webapp config Set](/cli/azure/webapp/deployment#az_webapp_deployment_list_publishing_profiles) con el argumento `--ftps-state`.

```azurecli-interactive
az webapp config set --name <app-name> --resource-group <group-name> --ftps-state FtpsOnly
```

Los valores posibles para `--ftps-state` son `AllAllowed` (FTP y FTPS habilitados), `Disabled` (FTP y FTPS deshabilitados) y `FtpsOnly` (solo FTPS).

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Ejecute el comando [Set-AzWebApp](/powershell/module/az.websites/set-azwebapp) con el parámetro `-FtpsState`.

```azurepowershell-interactive
Set-AzWebApp -Name <app-name> -ResourceGroupName <group-name> -FtpsState FtpsOnly
```

Los valores posibles para `--ftps-state` son `AllAllowed` (FTP y FTPS habilitados), `Disabled` (FTP y FTPS deshabilitados) y `FtpsOnly` (solo FTPS).

-----

[!INCLUDE [What happens to my app during deployment?](../../includes/app-service-deploy-atomicity.md)]

## <a name="troubleshoot-ftp-deployment"></a>Solución de problemas de implementación de FTP

- [¿Cómo se solucionan los problemas de implementación de FTP?](#how-can-i-troubleshoot-ftp-deployment)
- [No puedo usar FTP y publicar mi código. ¿Cómo se resuelve este problema?](#im-not-able-to-ftp-and-publish-my-code-how-can-i-resolve-the-issue)
- [¿Cómo me conecto a FTP en Azure App Service con el modo pasivo?](#how-can-i-connect-to-ftp-in-azure-app-service-via-passive-mode)

#### <a name="how-can-i-troubleshoot-ftp-deployment"></a>¿Cómo se solucionan los problemas de implementación de FTP?

El primer paso para solucionar los problemas de implementación de FTP es aislar los de implementación de los de la aplicación en tiempo de ejecución.

Un problema de implementación suele terminar en la ausencia de archivos o en la implementación de archivos incorrectos en la aplicación. Puede solucionar el problema al investigar la implementación FTP o seleccionar una ruta de implementación alternativa (como el control de código fuente).

Los problemas de aplicación en tiempo de ejecución suelen provocar la implementación del conjunto de archivos correctos en la aplicación, pero un comportamiento incorrecto de esta. Esto se puede solucionar si nos centramos en el comportamiento del código en tiempo de ejecución e investigamos las rutas de acceso con error concretas.

Para determinar un problema de implementación o de tiempo de ejecución, consulte [Deployment vs. runtime issues](https://github.com/projectkudu/kudu/wiki/Deployment-vs-runtime-issues) (Problemas de implementación frente a los de tiempo de ejecución).

#### <a name="im-not-able-to-ftp-and-publish-my-code-how-can-i-resolve-the-issue"></a>No puedo usar FTP y publicar mi código. ¿Cómo se resuelve este problema?
Compruebe que ha escrito el [nombre de host](#get-ftps-endpoint) y las [credenciales](#get-deployment-credentials) correctos. Compruebe también que los siguientes puertos FTP de la máquina no estén bloqueados por firewall:

- Puerto de conexión de control de FTP:
- Puerto de conexión de datos de FTP: 989, 10001-10300
 
#### <a name="how-can-i-connect-to-ftp-in-azure-app-service-via-passive-mode"></a>¿Cómo me conecto a FTP en Azure App Service con el modo pasivo?
Azure App Service permite la conexión activa y pasiva. Se recomienda el modo pasivo, ya que las máquinas de implementación suelen estar protegidas por firewall (del sistema operativo o como parte de una red particular o profesional). Consulte un [ejemplo en la documentación de WinSCP](https://winscp.net/docs/ui_login_connection). 

## <a name="more-resources"></a>Más recursos

* [Implementación de Git local en Azure App Service](deploy-local-git.md)
* [Credenciales de implementación de Azure App Service](deploy-configure-credentials.md)
* [Ejemplo: Creación de una aplicación web e implementación de archivos con FTP (CLI de Azure)](./scripts/cli-deploy-ftp.md).
* [Ejemplo: Carga de archivos a una aplicación web con FTP (PowerShell)](./scripts/powershell-deploy-ftp.md).
