---
title: Descubra las propiedades de los recursos
description: Describe cómo buscar propiedades de recursos.
ms.topic: conceptual
ms.date: 06/10/2020
ms.openlocfilehash: aaa7bb13f49c1c742e44a255d4bbdbd2b1730939
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108736692"
---
# <a name="discover-resource-properties"></a>Descubra las propiedades de los recursos

Antes de crear las plantillas de Resource Manager, debe comprender qué tipos de recursos están disponibles y qué valores usar en la plantilla. En este artículo se muestran algunas maneras en las que puede buscar las propiedades que incluir en la plantilla.

## <a name="find-resource-provider-namespaces"></a>Buscar espacios de nombres del proveedor de recursos

Los recursos de una plantilla de Resource Manager se definen con un espacio de nombres de proveedores de recursos y un tipo de recurso. Por ejemplo, Microsoft.Storage/storageAccounts es el nombre completo del tipo de recurso de la cuenta de almacenamiento. Microsoft.Storage es el espacio de nombres. Si aún no conoce los espacios de nombres de los tipos de recursos que quiere usar, consulte [Proveedores de recursos para los servicios de Azure](../management/azure-services-resource-providers.md).

![Asignación de espacios de nombres del proveedor de recursos de Resource Manager](./media/view-resources/resource-provider-namespace-and-azure-service-mapping.png)

## <a name="export-templates"></a>Exportar plantillas

La forma más fácil de obtener las propiedades de la plantilla para los recursos existentes es exportar la plantilla. Para más información, consulte [Exportación de uno y varios recursos a la plantilla en Azure Portal](./export-template-portal.md).

## <a name="use-resource-manager-tools-extension"></a>Uso de la extensión de herramientas de Resource Manager

Visual Studio Code y la extensión de herramientas de Azure Resource Manager le ayudan a ver exactamente qué propiedades son necesarias para cada tipo de recurso. Proporcionan IntelliSense y fragmentos de código que simplifican la forma de definir un recurso en la plantilla. Para más información, consulte [Inicio rápido: Creación de plantillas de Azure Resource Manager con Visual Studio Code](./quickstart-create-templates-use-visual-studio-code.md#add-an-azure-resource).

La captura de pantalla siguiente muestra un recurso de la cuenta de almacenamiento que se agrega a una plantilla:

![Fragmentos de código de extensión de herramientas de Resource Manager](./media/view-resources/resource-manager-tools-extension-snippets.png)

La extensión también proporciona una lista de opciones para las propiedades de configuración.

![Valores configurables de extensión de herramientas de Resource Manager](./media/view-resources/resource-manager-tools-extension-configurable-properties.png)

## <a name="use-template-reference"></a>Uso de referencia de plantilla

La referencia de plantilla de Azure Resource Manager es el recurso más completo para el esquema de plantilla. Puede encontrar las versiones de API, el formato de plantilla y la información de propiedades.

1. Examine [Referencia de plantilla de Azure Resource Manager](/azure/templates/).
1. En el panel de navegación izquierdo, seleccione **Almacenamiento** y, después, seleccione **Todos los recursos**. La página Todos los recursos resume los tipos de recursos y las versiones.

    ![versiones de recursos de referencia de plantilla](./media/view-resources/resource-manager-template-reference-resource-versions.png)

    Si conoce el tipo de recurso, puede ir directamente a esta página en el siguiente formato de dirección URL: `https://docs.microsoft.com/azure/templates/{provider-namespace}/{resource-type}`.

1. Seleccione la versión más reciente. Se recomienda usar la versión de API más reciente.

    La sección **formato de plantilla** enumera todas las propiedades de la cuenta de almacenamiento. **sku** está en la lista:

    ![Formato cuenta de almacenamiento de referencia de la plantilla](./media/view-resources/resource-manager-template-reference-storage-account-sku.png)

    Desplácese hacia abajo para ver el **objeto SKU** en la sección **Valores de propiedad**. En el artículo se muestran los valores permitidos para el nombre de SKU:

    ![Valores de SKU de la cuenta de almacenamiento de referencia de la plantilla](./media/view-resources/resource-manager-template-reference-storage-account-sku-values.png)

    Al final de la página, en la sección **Plantillas de inicio rápido** se enumeran algunas plantillas de inicio rápido de Azure que contienen el tipo de recurso:

    ![plantillas de inicio rápido de la cuenta de almacenamiento de referencia de plantilla](./media/view-resources/resource-manager-template-reference-quickstart-templates.png)

La referencia de plantilla se vincula desde cada uno de los sitios de documentación del servicio de Azure.  Por ejemplo, el [sitio de documentación de Key Vault](../../key-vault/general/overview.md):

![Referencia de plantilla de Resource Manager Key Vault](./media/view-resources/resource-manager-template-reference-key-vault.png)

## <a name="use-resource-explorer"></a>Usar el explorador de recursos

El explorador de recursos se incrusta en Azure Portal. Antes de usar este método, necesita una cuenta de almacenamiento. Si no la tiene, seleccione el siguiente botón para crear una:

[![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.storage%2Fstorage-account-create%2Fazuredeploy.json)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el cuadro de búsqueda, introduzca el **explorador de recursos** y, luego, seleccione **Explorador de recursos**.

    ![Captura de pantalla que muestra la búsqueda del Explorador de recursos en Azure Portal.](./media/view-resources/azure-portal-resource-explorer.png)

1. En la izquierda, expanda **Suscripciones** y, después, expanda su suscripción de Azure. Puede encontrar la cuenta de almacenamiento en **proveedores** o **ResourceGroups**.

    ![Explorador de recursos de Azure Portal](./media/view-resources/azure-portal-resource-explorer-home.png)

    - **Proveedores**: expanda **Proveedores** -> **Microsoft.Storage** -> **storageAccounts** y, luego, seleccione la cuenta de almacenamiento.
    - **ResourceGroups**: seleccione el grupo de recursos que contiene la cuenta de almacenamiento, seleccione **Recursos** y, después, seleccione la cuenta de almacenamiento.

    A la derecha, verá la configuración de SKU para la cuenta de almacenamiento existente, similar a:

    ![SKU de Azure Portal de la cuenta de almacenamiento Explorador de recursos](./media/view-resources/azure-portal-resource-explorer-sku.png)

## <a name="use-resourcesazurecom"></a>Uso de Resources.azure.com

Resources.azure.com es un sitio web público al que puede tener acceso cualquier usuario con una suscripción de Azure. Está en versión preliminar.  Considere la posibilidad de usar el [Explorador de recursos](#use-resource-explorer) en su lugar. Esta herramienta proporciona estas funcionalidades:

- Encontrar las API de administración de recursos de Azure.
- Obtener la documentación de la API y la información del esquema.
- Realizar llamadas API directamente en las suscripciones de Azure.

Para demostrar cómo se recupera la información de esquema mediante esta herramienta, se necesita una cuenta de almacenamiento. Si no la tiene, seleccione el siguiente botón para crear una:

[![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.storage%2Fstorage-account-create%2Fazuredeploy.json)

1. Vaya a [Resources.azure.com](https://resources.azure.com/). La herramienta tarda unos minutos en obtener el panel izquierdo.
1. Selección de **suscripciones**.

    ![asiresource.azure.com api mapping](./media/view-resources/resources-azure-com-api-mapping.png)

    El nodo de la izquierda coincide con la llamada API de la derecha. Puede realizar la llamada de API con el botón **Obtener**.
1. En la izquierda, expanda **Suscripciones** y, después, expanda su suscripción de Azure. Puede encontrar la cuenta de almacenamiento en **proveedores** o **ResourceGroups**.

    - **Proveedores**: expanda **Proveedores** -> **Microsoft.Storage** -> **storageAccounts** y vaya a la cuenta de almacenamiento.
    - **ResourceGroups**: seleccione el grupo de recursos que contiene la cuenta de almacenamiento y, después, seleccione **Recursos**.

    A la derecha, verá la configuración de sku para la cuenta de almacenamiento existente similar a:

    ![SKU de Azure Portal de la cuenta de almacenamiento Explorador de recursos](./media/view-resources/azure-portal-resource-explorer-sku.png)

## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió a buscar información de esquema de plantilla. Para obtener más información sobre la creación de plantillas de Resource Manager, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](./template-syntax.md).
