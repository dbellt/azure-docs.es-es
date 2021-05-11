---
title: Creación de su cuenta de Azure Maps mediante una plantilla de Azure Resource Manager (ARM) en Azure Maps
description: Obtenga información sobre cómo crear una cuenta de Azure Maps mediante una plantilla de Azure Resource Manager.
author: philmea
ms.author: philmea
ms.date: 04/27/2021
ms.topic: how-to
ms.service: azure-maps
ms.openlocfilehash: a2434494a769fb9a4695e698b3d9215dffc18974
ms.sourcegitcommit: f6b76df4c22f1c605682418f3f2385131512508d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108326874"
---
# <a name="create-your-azure-maps-account-using-an-arm-template"></a>Creación de su cuenta de Azure Maps mediante una plantilla de ARM

Puede crear su cuenta de Azure Maps mediante una plantilla de Azure Resource Manager (ARM). Una vez que tenga una cuenta, puede implementar las API en su sitio web o aplicación móvil.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-maps-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo:

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-maps-create/).

:::code language="json" source="~/quickstart-templates/101-maps-create/azuredeploy.json":::

El recurso de cuenta de Azure Maps se define en esta plantilla:

* [**Microsoft.Maps/accounts**](/azure/templates/microsoft.maps/accounts): cree una cuenta de Azure Maps.

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir una plantilla. La plantilla crea una cuenta de Azure Maps.

    [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-maps-create%2Fazuredeploy.json)

2. Seleccione o escriba los siguientes valores.

    ![Portal de implementación de plantilla de ARM](./media/how-to-create-template/create-account-using-template-portal.png)

    A menos que se especifique otra cosa, utilice el valor predeterminado para crear la cuenta de Azure Maps.

    * **Suscripción**: seleccione una suscripción de Azure.
    * **Grupo de recursos**: seleccione **Crear nuevo**, escriba un nombre único para el grupo de recursos y, a continuación, haga clic en **Aceptar**.
    * **Ubicación**: seleccione una ubicación. Por ejemplo, **Oeste de EE. UU. 2**.
    * **Nombre de cuenta**: escriba un nombre para la cuenta de Azure Maps, que debe ser único globalmente.
    * **Plan de tarifa**: seleccione el plan de tarifa adecuado, el valor predeterminado de la plantilla es S0.

3. Seleccione **Revisar + crear**. 
4. Confirme la configuración en la página de revisión y haga clic en **Crear**. Una vez implementada correctamente la instancia de Azure Maps, recibirá una notificación:

    ![Notificación del portal de implementación de plantilla de ARM](./media/how-to-create-template/resource-manager-template-portal-deployment-notification.png)

Azure Portal se usa para implementar la plantilla. También puede usar Azure PowerShell, la CLI de Azure o la API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../azure-resource-manager/templates/deploy-powershell.md).

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Puede usar Azure Portal para comprobar su cuenta de Azure Maps y ver las claves. También puede usar el siguiente script de la CLI de Azure para mostrar las claves de cuenta.

```azurecli-interactive
az maps account keys list --name MyMapsAccount --resource-group MyResourceGroup
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, elimine el grupo de recursos, que también elimina la cuenta de Azure Maps. Para eliminar el grupo de recursos mediante la CLI de Azure:

```azurecli-interactive
az group delete --name MyResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Maps y Azure Resource Manager, continúe con los artículos siguientes.

- Creación de una [aplicación de demostración](quick-demo-map-app.md) de Azure Maps
- Más información sobre [plantillas de ARM](../azure-resource-manager/templates/overview.md)