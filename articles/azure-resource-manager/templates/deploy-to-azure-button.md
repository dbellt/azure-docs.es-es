---
title: Botón Implementación en Azure
description: Use el botón para implementar plantillas de Azure Resource Manager desde un repositorio de GitHub.
ms.topic: conceptual
ms.date: 05/05/2021
ms.openlocfilehash: 77d23852941177aca35a582f64b21bfac02d7942
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108736908"
---
# <a name="use-a-deployment-button-to-deploy-templates-from-github-repository"></a>Usar un botón de implementación para implementar plantillas desde el repositorio de GitHub

En este artículo se describe cómo usar el botón **Implementar en Azure** para implementar plantillas desde un repositorio de GitHub. Puede agregar el botón directamente al archivo _README.md_ en el repositorio de GitHub. O bien, puede agregar el botón a una página web que haga referencia al repositorio.

El ámbito de implementación viene determinado por el esquema de plantilla. Para más información, consulte:

- [resource groups](deploy-to-resource-group.md)
- [subscriptions](deploy-to-subscription.md)
- [grupos de administración](deploy-to-management-group.md)
- [tenants](deploy-to-tenant.md)

## <a name="use-common-image"></a>Uso de una imagen común

Para agregar el botón a su página web o repositorio, use la siguiente imagen:

```markdown
![Deploy to Azure](https://aka.ms/deploytoazurebutton)
```

```html
<img src="https://aka.ms/deploytoazurebutton"/>
```

La imagen aparece como:

![Botón Implementación en Azure](https://aka.ms/deploytoazurebutton)

## <a name="create-url-for-deploying-template"></a>Crear dirección URL para implementar una plantilla

Para crear la dirección URL de la plantilla, comience con la dirección URL original de la plantilla en el repositorio. Para ver la dirección URL sin formato, seleccione **Sin formato**.

:::image type="content" source="./media/deploy-to-azure-button/select-raw.png" alt-text="Seleccione sin procesar":::

El formato de la dirección URL es:

```html
https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.storage/storage-account-create/azuredeploy.json
```

A continuación, convierta la dirección URL en un valor con codificación URL. Puede usar un codificador en línea o ejecutar un comando. En el siguiente ejemplo de PowerShell se muestra cómo codificar un valor como una dirección URL.

```powershell
$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.storage/storage-account-create/azuredeploy.json"
[uri]::EscapeDataString($url)
```

La dirección URL de ejemplo tiene el siguiente valor cuando se codifica como una URL.

```html
https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.storage%2Fstorage-account-create%2Fazuredeploy.json
```

Cada vínculo comienza con la misma dirección URL base:

```html
https://portal.azure.com/#create/Microsoft.Template/uri/
```

Agregue el vínculo de la plantilla con codificación URL al final de la dirección URL base.

```html
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.storage%2Fstorage-account-create%2Fazuredeploy.json
```

Ya tiene la dirección URL completa del vínculo.

[!INCLUDE [Deploy templates in private GitHub repo](../../../includes/resource-manager-private-github-repo-templates.md)]

Si usa [Git con Azure Repos](/azure/devops/repos/git/) en lugar de un repositorio de GitHub, puede seguir usando el botón **Implementar en Azure**. Asegúrese de que el repositorio es público. Use la [operación Elementos](/rest/api/azure/devops/git/items/get) para obtener la plantilla. La solicitud debería tener el siguiente formato:

```http
https://dev.azure.com/{organization-name}/{project-name}/_apis/git/repositories/{repository-name}/items?scopePath={url-encoded-path}&api-version=6.0
```

Codifique esta dirección URL de solicitud.

## <a name="create-deploy-to-azure-button"></a>Crear el botón Implementar en Azure

Por último, coloque el vínculo y la imagen juntos.

Para agregar el botón con Markdown en el archivo _README.md_ en el repositorio de GitHub o en una página web, use:

```markdown
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.storage%2Fstorage-account-create%2Fazuredeploy.json)
```

Para HTML, use:

```html
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.storage%2Fstorage-account-create%2Fazuredeploy.json" target="_blank">
  <img src="https://aka.ms/deploytoazurebutton"/>
</a>
```

En el caso de Git con el repositorio de Azure, el botón tiene el siguiente formato:

```markdown
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fdev.azure.com%2Forgname%2Fprojectname%2F_apis%2Fgit%2Frepositories%2Freponame%2Fitems%3FscopePath%3D%2freponame%2fazuredeploy.json%26api-version%3D6.0)
```

## <a name="deploy-the-template"></a>Implementación de la plantilla

Para probar la solución completa, seleccione el botón siguiente:

[![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.storage%2Fstorage-account-create%2Fazuredeploy.json)

El portal muestra un panel que le permite proporcionar fácilmente los valores de parámetros. Los parámetros se rellenan previamente con los valores predeterminados de la plantilla. El nombre del parámetro con mayúsculas y minúsculas Camel, *storageAccountType*, definido en la plantilla se convierte en una cadena separada por espacios cuando se muestra en el portal.

![Usar el portal para realizar la implementación](./media/deploy-to-azure-button/portal.png)

## <a name="next-steps"></a>Pasos siguientes

- Para información sobre las plantillas, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de ARM](template-syntax.md).
