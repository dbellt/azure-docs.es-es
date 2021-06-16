---
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 08/12/2019
ms.author: cephalin
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e32f6bd93d708c2fc58ac037160e8e6f98a32114
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110721938"
---
## <a name="deploy-zip-file-with-rest-apis"></a><a name="rest"></a>Implementación de un archivo ZIP con las API REST 

Puede usar el [servicio de implementación de las API de REST](https://github.com/projectkudu/kudu/wiki/REST-API) para implementar el archivo .zip en su aplicación en Azure. Para ello, envíe una solicitud POST a https://<app_name>.scm.azurewebsites.net/api/zipdeploy. La solicitud POST debe contener el archivo .zip en el cuerpo del mensaje. Las credenciales de implementación de la aplicación se proporcionan en la solicitud mediante la autenticación de HTTP BASIC. Para obtener más información, consulte la [referencia de implementación para la inserción del archivo .zip](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file). 

Para la autenticación básica HTTP, necesita las credenciales de implementación de App Service. Para ver cómo establecer sus credenciales de implementación, consulte [Establecimiento y restablecimiento de credenciales de nivel de usuario](../articles/app-service/deploy-configure-credentials.md#userscope).

### <a name="with-curl"></a>Con cURL

En el ejemplo siguiente se usa la herramienta cURL para implementar un archivo .zip. Reemplace los marcadores de posición `<deployment_user>`, `<zip_file_path>` y `<app_name>`. Cuando cURL se lo solicite, escriba la contraseña.

```bash
curl -X POST -u <deployment_user> --data-binary @"<zip_file_path>" https://<app_name>.scm.azurewebsites.net/api/zipdeploy
```

Esta solicitud desencadena la implementación de inserción desde el archivo .zip cargado. Puede revisar las implementaciones actuales y pasadas mediante el punto de conexión `https://<app_name>.scm.azurewebsites.net/api/deployments`, tal como se muestra en el siguiente ejemplo de cURL. Una vez más, reemplace `<app_name>` por el nombre de la aplicación y `<deployment_user>` por el nombre de usuario de las credenciales de implementación.

```bash
curl -u <deployment_user> https://<app_name>.scm.azurewebsites.net/api/deployments
```

### <a name="with-powershell"></a>Con PowerShell

En el ejemplo siguiente se usa [Publish-AzWebapp](/powershell/module/az.websites/publish-azwebapp) para cargar el archivo .zip. Reemplace los marcadores de posición `<group-name>`, `<app-name>` y `<zip-file-path>`.

```powershell
Publish-AzWebapp -ResourceGroupName <group-name> -Name <app-name> -ArchivePath <zip-file-path>
```

Esta solicitud desencadena la implementación de inserción desde el archivo .zip cargado. 

Para revisar las implementaciones actuales y pasadas, ejecute los siguientes comandos. De nuevo, reemplace los marcadores `<deployment-user>`, `<deployment-password>` y `<app-name>`.

```bash
$username = "<deployment-user>"
$password = "<deployment-password>"
$apiUrl = "https://<app-name>.scm.azurewebsites.net/api/deployments"
$base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(("{0}:{1}" -f $username, $password)))
$userAgent = "powershell/1.0"
Invoke-RestMethod -Uri $apiUrl -Headers @{Authorization=("Basic {0}" -f $base64AuthInfo)} -UserAgent $userAgent -Method GET
```
