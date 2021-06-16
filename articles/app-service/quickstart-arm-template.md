---
title: Creación de una aplicación de App Service mediante el uso de una plantilla de Azure Resource Manager
description: Cree su primera aplicación en Azure App Service en segundos mediante una plantilla de Azure Resource Manager (ARM), que es una de las muchas maneras de realizar una implementación en App Service.
author: msangapu-msft
ms.author: msangapu
ms.assetid: 582bb3c2-164b-42f5-b081-95bfcb7a502a
ms.topic: quickstart
ms.date: 10/16/2020
ms.custom: subject-armqs
zone_pivot_groups: app-service-platform-windows-linux
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./quickstart-arm-template-uiex
ms.openlocfilehash: b74287a4324d7a82cf3f47a008a3809e3c5690be
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111892623"
---
# <a name="quickstart-create-app-service-app-using-an-arm-template"></a>Inicio rápido: Creación de una aplicación de App Service mediante una plantilla de ARM

Empiece a usar [Azure App Service](overview.md) mediante la implementación de una aplicación en la nube con una plantilla de Azure Resource Manager (ARM) y la [CLI de Azure](/cli/azure/get-started-with-azure-cli) en Cloud Shell. Dado que usa un nivel de App Service gratuito, completar este inicio rápido no supone ningún costo.

 [!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

Use el botón siguiente para realizar la implementación en **Linux**:

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.web%2Fapp-service-docs-linux%2Fazuredeploy.json)

Use el botón siguiente para realizar la implementación en **Windows**:

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.web%2Fapp-service-docs-windows%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="review-the-template"></a>Revisión de la plantilla

::: zone pivot="platform-windows"
La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/app-service-docs-windows). Implementa un plan y una aplicación de App Service en Windows. Es compatible con las aplicaciones de .NET Core, .NET Framework, PHP, Node.js y HTML estático. Para Java, consulte [Creación de aplicaciones Java](./quickstart-java.md).

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.web/app-service-docs-windows/azuredeploy.json":::

En la plantilla se definen dos recursos de Azure:

* [**Microsoft.Web/serverfarms**](/azure/templates/microsoft.web/serverfarms): permite crear un plan de App Service.
* [**Microsoft.Web/sites**](/azure/templates/microsoft.web/sites): permite crear una aplicación de App Service.

Esta plantilla contiene varios parámetros que están predefinidos para su comodidad. Consulte la tabla siguiente para ver los valores predeterminados de los parámetros y sus descripciones:

| Parámetros | Tipo    | Valor predeterminado                | Descripción |
|------------|---------|------------------------------|-------------|
| webAppName | string  | "webApp- **[`<uniqueString>`](../azure-resource-manager/templates/template-functions-string.md#uniquestring)** " | Nombre de la aplicación |
| ubicación   | string  | "[[resourceGroup().location](../azure-resource-manager/templates/template-functions-resource.md#resourcegroup)]" | Región de la aplicación |
| sku        | string  | "F1"                         | Tamaño de la instancia (F1 = nivel Gratis) |
| language   | string  | ".net"                       | Pila de lenguajes de programación (.net, php, node, html) |
| helloWorld | boolean | False                        | True = implementa la aplicación "Hola mundo" |
| repoUrl    | string  | " "                          | Repositorio de Git externo (opcional) |
::: zone-end
::: zone pivot="platform-linux"
La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/app-service-docs-linux). Implementa un plan y una aplicación de App Service en Linux. Es compatible con todos los lenguajes de programación que se admiten en App Service.

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.web/app-service-docs-linux/azuredeploy.json":::

En la plantilla se definen dos recursos de Azure:

* [**Microsoft.Web/serverfarms**](/azure/templates/microsoft.web/serverfarms): permite crear un plan de App Service.
* [**Microsoft.Web/sites**](/azure/templates/microsoft.web/sites): permite crear una aplicación de App Service.

Esta plantilla contiene varios parámetros que están predefinidos para su comodidad. Consulte la tabla siguiente para ver los valores predeterminados de los parámetros y sus descripciones:

| Parámetros | Tipo    | Valor predeterminado                | Descripción |
|------------|---------|------------------------------|-------------|
| webAppName | string  | "webApp- **[`<uniqueString>`](../azure-resource-manager/templates/template-functions-string.md#uniquestring)** " | Nombre de la aplicación |
| ubicación   | string  | "[[resourceGroup().location](../azure-resource-manager/templates/template-functions-resource.md#resourcegroup)]" | Región de la aplicación |
| sku        | string  | "F1"                         | Tamaño de la instancia (F1 = nivel Gratis) |
| linuxFxVersion   | string  | "DOTNETCORE&#124;3.0        | "Pila de lenguajes de programación &#124; Versión" |
| repoUrl    | string  | " "                          | Repositorio de Git externo (opcional) |

---
::: zone-end

## <a name="deploy-the-template"></a>Implementación de la plantilla

Aquí se usa la CLI de Azure para implementar la plantilla. También puede usar Azure Portal, Azure PowerShell y la API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../azure-resource-manager/templates/deploy-powershell.md).

El siguiente código crea un grupo de recursos, un plan de App Service y una aplicación web. Se han establecido un grupo de recursos, un plan de App Service y una ubicación predeterminados. Reemplace `<app-name>` por un nombre de aplicación único global (los caracteres válidos son `a-z`, `0-9` y `-`).

::: zone pivot="platform-windows"
Ejecute el código siguiente para implementar una aplicación de .NET Framework en Windows.

```azurecli-interactive
az group create --name myResourceGroup --location "southcentralus" &&
az deployment group create --resource-group myResourceGroup \
--parameters language=".net" helloWorld="true" webAppName="<app-name>" \
--template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.web/app-service-docs-windows/azuredeploy.json"
::: zone-end
::: zone pivot="platform-linux"
Run the code below to create a Python app on Linux.

```azurecli-interactive
az group create --name myResourceGroup --location "southcentralus" &&
az deployment group create --resource-group myResourceGroup --parameters webAppName="<app-name>" linuxFxVersion="PYTHON|3.7" \
--template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.web/app-service-docs-linux/azuredeploy.json"
```

Para implementar una pila de lenguajes diferente, actualice `linuxFxVersion` con los valores adecuados. A continuación se muestran ejemplos. Para mostrar las versiones actuales, ejecute el siguiente comando en Cloud Shell: `az webapp config show --resource-group myResourceGroup --name <app-name> --query linuxFxVersion`

| Idioma    | Ejemplo:                                               |
|-------------|------------------------------------------------------|
| **.NET**    | linuxFxVersion="DOTNETCORE&#124;3.0"                 |
| **PHP**     | linuxFxVersion="PHP&#124;7.4"                        |
| **Node.js** | linuxFxVersion="NODE&#124;10.15"                     |
| **Java**    | linuxFxVersion="JAVA&#124;1.8 &#124;TOMCAT&#124;9.0" |
| **Python**  | linuxFxVersion="PYTHON&#124;3.7"                     |
| **Ruby**    | linuxFxVersion="RUBY&#124;2.6"                       |

---
::: zone-end

> [!NOTE]
> Puede encontrar [aquí más ejemplos de plantillas de Azure App Service](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Sites).

## <a name="validate-the-deployment"></a>Validación de la implementación

Vaya a `http://<app_name>.azurewebsites.net/` y compruebe que se ha creado.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite, [elimine el grupo de recursos](../azure-resource-manager/management/delete-resource-group.md?tabs=azure-portal#delete-resource-group).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Implementación desde Git local](deploy-local-git.md)

> [!div class="nextstepaction"]
> [ASP.NET Core con SQL Database](tutorial-dotnetcore-sqldb-app.md)

> [!div class="nextstepaction"]
> Python con Postgres

> [!div class="nextstepaction"]
> [PHP con MySQL](tutorial-php-mysql-app.md)

> [!div class="nextstepaction"]
> [Conexión a una base de datos de Azure SQL con Java](../azure-sql/database/connect-query-java.md?toc=%2fazure%2fjava%2ftoc.json)

> [!div class="nextstepaction"]
> [Asignación de un dominio personalizado](app-service-web-tutorial-custom-domain.md)
