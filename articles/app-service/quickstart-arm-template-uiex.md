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
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: bfdfad8af2d730caf0ac9e22ee815a6d59a6b6a3
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108074734"
---
# <a name="quickstart-create-app-service-app-using-an-arm-template"></a>Inicio rápido: Creación de una aplicación de App Service mediante una plantilla de ARM

Empiece a usar [Azure App Service](overview.md) mediante la implementación de una aplicación en la nube con una <abbr title="Un archivo JSON que define de forma declarativa uno o varios recursos de Azure y las dependencias entre los recursos implementados. La plantilla se puede usar para implementar los recursos de manera repetida y uniforme.">Plantilla ARM</abbr> y la [CLI de Azure](/cli/azure/get-started-with-azure-cli) en Cloud Shell. Dado que usa un nivel de App Service gratuito, completar este inicio rápido no supone ningún costo. <abbr title="En la sintaxis declarativa, se describe la implementación deseada sin escribir la secuencia de comandos de programación para crearla.">La plantilla usa sintaxis declarativa.</abbr>

 Si su entorno cumple los requisitos previos y ya está familiarizado con el uso de [plantillas de ARM](../azure-resource-manager/templates/overview.md), seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

::: zone pivot="platform-windows"
[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.web%2Fapp-service-docs-windows%2Fazuredeploy.json)
::: zone-end

::: zone pivot="platform-linux"
[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.web%2Fapp-service-docs-linux%2Fazuredeploy.json)
::: zone-end

<hr/>

## <a name="1-prepare-your-environment"></a>1. Preparación del entorno

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

<hr/>

## <a name="2-review-the-template"></a>2. Revisión de la plantilla

::: zone pivot="platform-windows"
La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-app-service-docs-windows). Implementa un plan y una aplicación de App Service en Windows.

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.web/app-service-docs-windows/azuredeploy.json":::

<details>
<summary>¿Qué recursos y parámetros se definen en la plantilla?</summary>

En la plantilla se definen dos recursos de Azure:

* [**Microsoft.Web/serverfarms**](/azure/templates/microsoft.web/serverfarms): permite crear un plan de App Service.
* [**Microsoft.Web/sites**](/azure/templates/microsoft.web/sites): permite crear una aplicación de App Service.

En la tabla siguiente se detallan y describen los parámetros predeterminados:

| Parámetros | Tipo    | Valor predeterminado                | Descripción |
|------------|---------|------------------------------|-------------|
| webAppName | string  | "webApp- **[`<uniqueString>`](../azure-resource-manager/templates/template-functions-string.md#uniquestring)** " | Nombre de la aplicación |
| ubicación   | string  | "[[resourceGroup().location](../azure-resource-manager/templates/template-functions-resource.md#resourcegroup)]" | Región de la aplicación |
| sku        | string  | "F1"                         | Tamaño de la instancia (F1 = nivel Gratis) |
| language   | string  | ".net"                       | Pila de lenguajes de programación (.net, php, node, html) |
| helloWorld | boolean | False                        | True = implementa la aplicación "Hola mundo" |
| repoUrl    | string  | " "                          | Repositorio de Git externo (opcional) |

---

</details>
::: zone-end
::: zone pivot="platform-linux"
La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-app-service-docs-linux). Implementa un plan y una aplicación de App Service en Windows.

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.web/app-service-docs-linux/azuredeploy.json":::

Esta plantilla incluye recursos y parámetros de Azure que se definen para su comodidad.

<details>
<summary>¿Qué recursos y parámetros se definen en la plantilla?</summary>

En la plantilla se definen dos recursos de Azure:

* [**Microsoft.Web/serverfarms**](/azure/templates/microsoft.web/serverfarms): permite crear un plan de App Service.
* [**Microsoft.Web/sites**](/azure/templates/microsoft.web/sites): permite crear una aplicación de App Service.

En la tabla siguiente se detallan y describen los parámetros predeterminados:

| Parámetros | Tipo    | Valor predeterminado                | Descripción |
|------------|---------|------------------------------|-------------|
| webAppName | string  | "webApp- **[`<uniqueString>`](../azure-resource-manager/templates/template-functions-string.md#uniquestring)** " | Nombre de la aplicación |
| ubicación   | string  | "[[resourceGroup().location](../azure-resource-manager/templates/template-functions-resource.md#resourcegroup)]" | Región de la aplicación |
| sku        | string  | "F1"                         | Tamaño de la instancia (F1 = nivel Gratis) |
| linuxFxVersion   | string  | "DOTNETCORE&#124;3.0        | "Pila de lenguajes de programación &#124; Versión" |
| repoUrl    | string  | " "                          | Repositorio de Git externo (opcional) |

---

</details>

::: zone-end

<hr/>

## <a name="3-deploy-the-template"></a>3. Implementación de la plantilla

::: zone pivot="platform-windows"
Ejecute el código siguiente para implementar una aplicación de .NET Framework en Windows mediante la CLI de Azure. 

Replace <abbr title="Los caracteres válidos son `a-z`, `0-9` y `-`.">`<app-name>`</abbr> con un nombre de aplicación único global. Para conocer otros <abbr title="También puede usar Azure Portal, Azure PowerShell y la API REST.">métodos de implementación</abbr>, consulte [Implementación de plantillas](../azure-resource-manager/templates/deploy-powershell.md). Puede encontrar [aquí más ejemplos de plantillas de Azure App Service](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Sites).

```azurecli-interactive
az group create --name myResourceGroup --location "southcentralus" &&
az deployment group create --resource-group myResourceGroup \
--parameters language=".net" helloWorld="true" webAppName="<app-name>" \
--template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.web/app-service-docs-windows/azuredeploy.json"
```
::: zone-end
::: zone pivot="platform-linux"
Ejecute el código siguiente para crear una aplicación de Python en Linux. 

Replace <abbr title="Los caracteres válidos son `a-z`, `0-9` y `-`.">`<app-name>`</abbr> con un nombre de aplicación único global.

```azurecli-interactive
az group create --name myResourceGroup --location "southcentralus" &&
az deployment group create --resource-group myResourceGroup --parameters webAppName="<app-name>" linuxFxVersion="PYTHON|3.7" \
--template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.web/app-service-docs-linux/azuredeploy.json"
```
::: zone-end

<details>
<summary>¿Qué hace el código?</summary>
<p>Los comandos llevan a cabo las siguientes acciones:</p>
<ul>
<li>Crear un grupo de recursos predeterminado. <abbr title="Un contenedor lógico para recursos relacionados de Azure que se pueden administrar como una unidad.">resource group</abbr>.</li>
<li>Crear un plan de App Service predeterminado. <abbr title="El plan especifica la ubicación, el tamaño y las características de la granja de servidores web que hospeda la aplicación.">Plan de App Service</abbr>.</li>
<li><a href="/cli/azure/webapp#az_webapp_create">Crear una <abbr title="La representación de la aplicación web, que contiene el código de la aplicación, los nombres de host DNS, los certificados y los recursos relacionados.">aplicación de App Service</abbr></a> con el nombre especificado.</li>
</ul>
</details>

::: zone pivot="platform-windows"
<details>
<summary>¿Cómo implementar una pila de lenguajes diferente?</summary>
Para implementar una pila de lenguajes diferente, actualice el <abbr title="Esta plantilla es compatible con las aplicaciones de .NET Core, .NET Framework, Node.js y HTML estático.">parámetro de lenguaje</abbr> con los valores adecuados. Para Java, consulte <a href="/azure/app-service/quickstart-java-uiex">Creación de aplicaciones Java</a>.

| Parámetros | Tipo    | Valor predeterminado                | Descripción |
|------------|---------|------------------------------|-------------|
| language   | string  | ".net"                       | Pila de lenguajes de programación (.net, php, node, html) |

---

</details>
::: zone-end
::: zone pivot="platform-linux"
<details>
<summary>¿Cómo implementar una pila de lenguajes diferente?</summary>
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

</details>
::: zone-end

<hr/>

## <a name="4-validate-the-deployment"></a>4. Validación de la implementación

Vaya a `http://<app_name>.azurewebsites.net/` y compruebe que se ha creado.

<hr/>

## <a name="5-clean-up-resources"></a>5. Limpieza de recursos

Cuando ya no necesite, [elimine el grupo de recursos](../azure-resource-manager/management/delete-resource-group.md?tabs=azure-portal#delete-resource-group).

<hr/>

## <a name="next-steps"></a>Pasos siguientes

- [Implementación desde Git local](deploy-local-git.md)
- [ASP.NET Core con SQL Database](tutorial-dotnetcore-sqldb-app.md)
- Python con Postgres
- [PHP con MySQL](tutorial-php-mysql-app.md)
- [Conexión a una base de datos de Azure SQL con Java](../azure-sql/database/connect-query-java.md?toc=%2fazure%2fjava%2ftoc.json)
- [Asignación de un dominio personalizado](app-service-web-tutorial-custom-domain-uiex.md)
