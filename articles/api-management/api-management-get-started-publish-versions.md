---
title: 'Tutorial: Publicación de versiones de la API con Azure API Management'
description: Siga los pasos de este tutorial para aprender a publicar varias versiones de la API en API Management.
author: vladvino
ms.service: api-management
ms.custom: mvc, devx-track-azurecli
ms.topic: tutorial
ms.date: 02/10/2021
ms.author: apimpm
ms.openlocfilehash: 5930979258372e4269c4d453ffbc5e0d46258088
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107483620"
---
# <a name="tutorial-publish-multiple-versions-of-your-api"></a>Tutorial: Publicación de varias versiones de la API 

A veces resulta poco práctico que todos los autores de llamadas que llaman a la API usen exactamente la misma versión. Si los autores de llamadas quieren actualizarla a una versión posterior, querrán hacerlo con un enfoque que sea fácil de comprender. Como se muestra en este tutorial, es posible proporcionar varias *versiones* en Azure API Management. 

Para obtener información general, consulte [Versiones y revisiones](https://azure.microsoft.com/blog/versions-revisions/).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Agregar una nueva versión a una API existente
> * Elegir un esquema de versión
> * Agregar la versión a un producto
> * Examinar el portal para desarrolladores para ver la versión

:::image type="content" source="media/api-management-getstarted-publish-versions/azure-portal.png" alt-text="Versión que se muestra en Azure portal":::

## <a name="prerequisites"></a>Prerequisites

+ Conocer la [terminología de API Management de Azure](api-management-terminology.md).
+ Complete el siguiente inicio rápido: [Creación de una instancia de Azure API Management](get-started-create-service-instance.md).
+ Además, completar el tutorial siguiente: [Importación y publicación de la primera API](import-and-publish.md).

## <a name="add-a-new-version"></a>Agregar una nueva versión

1. Vaya a la instancia de API Management en [Azure Portal](https://portal.azure.com).
1. Seleccione **API**.
1. Seleccione **Demo Conference API** en la lista de API. 
1. Seleccione el menú contextual ( **...** ) junto a **Demo Conference API**.
1. Seleccione **Agregar versión**.

:::image type="content" source="media/api-management-getstarted-publish-versions/add-version-menu.png" alt-text="Menú contextual de la API: agregar versión":::


> [!TIP]
> También se pueden habilitar las versiones cuando se crea una nueva API. En la pantalla **Agregar API**, seleccione **¿Definir versión de esta API?** .

## <a name="choose-a-versioning-scheme"></a>Elegir un esquema de control de versiones

En Azure API Management, elija cómo los autores de llamadas especifican la versión de la API seleccionando un *esquema de control de versiones*: **ruta, encabezado** o **cadena de consulta**. En el ejemplo siguiente, la *ruta de acceso* se utiliza como esquema del control de versiones.

Escriba los valores de la tabla siguiente. Luego, seleccione **Crear** para crear la versión.

:::image type="content" source="media/api-management-getstarted-publish-versions/add-version.png" alt-text="Ventana Agregar versión":::



|Configuración   |Value  |Descripción  |
|---------|---------|---------|
|**Nombre**     |  *demo-conference-api-v1*       |  Nombre único de su instancia de API Management.<br/><br/>Dado que una versión es en realidad una nueva API basada en una [revisión](api-management-get-started-revise-api.md) de la API, este valor es el nombre de la nueva API.   |
|**Esquema de control de versiones**     |  **Path**       |  La forma en que los autores de llamadas especifican la versión de la API.     |
|**Identificador de versión**     |  *v1*       |  Indicador específico del esquema de la versión. Para **Ruta de acceso**, el sufijo de la ruta de acceso de la URL de la API. <br/><br/> Si selecciona **Encabezado** o **Cadena de consulta**, proporcione un valor más: el nombre del parámetro de encabezado o de la cadena de consulta.<br/><br/> Se muestra un ejemplo de uso.        |
|**Productos**     |  **Sin límite**       |  Opcionalmente, uno o varios productos a los que está asociada la versión de la API. Para publicar la API, debe asociarla a un producto. También puede [agregar la versión a un producto](#add-the-version-to-a-product) más adelante.      |

Después de crear la versión, ahora aparece debajo de **Demo Conference API** en la lista de API. Ahora verá dos API: **Original** y **v1**.

![Versiones que aparecen en una API en Azure Portal](media/api-management-getstarted-publish-versions/version-list.png)

Ahora puede modificar y configurar **v1** como una API independiente de **Original**. Los cambios en una versión no afectan a otra.

> [!Note]
> Si agrega una versión a una API sin control de versiones, se creará automáticamente una API **Original**. Esta versión responde en la dirección URL predeterminada. La creación de una versión Original garantiza que los autores de llamadas no sufran ninguna interrupción a causa del proceso de adición de la versión. Si crea una API con las versiones habilitadas desde el principio, no se creará un original.

## <a name="add-the-version-to-a-product"></a>Agregar la versión a un producto

Para que quienes realizan las llamadas vean la versión nueva, debe agregarse a un *producto*. Si aún no ha agregado la versión a un producto, puede hacerlo en cualquier momento.

Por ejemplo, para agregar la versión al producto **Ilimitado**:
1. Vaya a la instancia de API Management en Azure Portal.
1. Seleccione **Productos** > **Ilimitado** > **API** >  **+ Agregar**.
1. Seleccione **Demo Conference API**, versión **v1**.
1. Haga clic en **Seleccionar**.

:::image type="content" source="media/api-management-getstarted-publish-versions/08-add-multiple-versions-03-add-version-product.png" alt-text="Agregar versión al producto":::

## <a name="use-version-sets"></a>Uso de conjuntos de versiones

Al crear varias versiones, Azure Portal crea un *conjunto de versiones*, que representa un conjunto de versiones para una única API lógica. Seleccione el nombre de una API que tenga varias versiones. Azure Portal muestra su **conjunto de versiones**. Puede personalizar el **Nombre** y la **Descripción** de un conjunto virtual.

Puede interactuar directamente con los conjuntos de versiones mediante la CLI de Azure:

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

Para ver todos los conjuntos de versiones, ejecute el comando [az apim api versionset list](/cli/azure/apim/api/versionset#az_apim_api_versionset_list):

```azurecli
az apim api versionset list --resource-group apim-hello-word-resource-group \
    --service-name apim-hello-world --output table
```

Cuando Azure Portal crea un conjunto de versiones, asigna un nombre alfanumérico, que aparece en la columna **Nombre** de la lista. Use este nombre en otros comandos de la CLI de Azure.

Para ver detalles acerca de un conjunto de versiones, ejecute el comando [az apim api versionset show](/cli/azure/apim/api/versionset#az_apim_api_versionset_show):

```azurecli
az apim api versionset show --resource-group apim-hello-word-resource-group \
    --service-name apim-hello-world --version-set-id 00000000000000000000000
```

Para más información sobre las suscripciones, consulte [Versiones de Azure API Management](api-management-versions.md#how-versions-are-represented).

## <a name="browse-the-developer-portal-to-see-the-version"></a>Examinar el portal para desarrolladores para ver la versión

Si ha probado el [portal para desarrolladores](api-management-howto-developer-portal-customize.md), puede ver las versiones de la API allí.

1. Seleccione **Portal para desarrolladores** en el menú superior.
2. Seleccione **API** y, después, **Demo Conference API**.
3. Debe ver una lista desplegable con varias versiones junto al nombre de la API.
4. Seleccione **v1**.
5. Fíjese en la **Dirección URL de la solicitud** de la primera operación de la lista. Refleja que la ruta de acceso de la dirección URL de la API incluye **v1**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Agregar una nueva versión a una API existente
> * Elegir un esquema de versión 
> * Agregar la versión a un producto
> * Examinar el portal para desarrolladores para ver la versión

Avance hasta el siguiente tutorial:

> [!div class="nextstepaction"]
> [Personalización del estilo de las páginas del portal para desarrolladores](api-management-howto-developer-portal-customize.md)
