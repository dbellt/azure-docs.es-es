---
title: 'Inicio rápido: Creación de un registro con replicación geográfica (plantilla de Azure Resource Manager)'
description: Aprenda a crear una instancia de Azure Container Registry mediante una plantilla de Azure Resource Manager.
services: azure-resource-manager
author: dlepow
ms.author: danlep
ms.date: 10/06/2020
ms.topic: quickstart
ms.service: azure-resource-manager
ms.custom:
- subject-armqs
- mode-arm
ms.openlocfilehash: 044c5abe7b44ebc4b9d0cd172f14a8cd1a648b2a
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107951583"
---
# <a name="quickstart-create-a-geo-replicated-container-registry-by-using-an-arm-template"></a>Inicio rápido: Creación de un registro de contenedor con replicación geográfica mediante una plantilla de Resource Manager

En este inicio rápido se muestra cómo crear una instancia de Azure Container Registry mediante una plantilla de Azure Resource Manager. La plantilla configura un registro con [replicación geográfica](container-registry-geo-replication.md), que sincroniza automáticamente el contenido del registro entre varias regiones de Azure. La replicación geográfica permite el acceso próximo a la red a imágenes de implementaciones regionales y, al mismo tiempo, permite la administración individual. Es una característica del nivel de servicio del registro [Premium](container-registry-skus.md).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.containerregistry%2Fcontainer-registry-geo-replication%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-container-registry-geo-replication/). La plantilla configura un registro y una réplica regional adicional.

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.containerregistry/container-registry-geo-replication/azuredeploy.json":::

Los recursos siguientes se definen en la plantilla:

* **[Microsoft.ContainerRegistry/registries](/azure/templates/microsoft.containerregistry/registries)** : cree una instancia de Azure Container Registry.
* **[Microsoft.ContainerRegistry/registries/replications](/azure/templates/microsoft.containerregistry/registries/replications)** : crea una réplica del registro de contenedor.

Encontrará más ejemplos de plantillas de Azure Container Registry en la [galería de plantillas de inicio rápido](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Containerregistry&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Implementación de la plantilla

 1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir una plantilla.

    [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.containerregistry%2Fcontainer-registry-geo-replication%2Fazuredeploy.json)

 1. Seleccione o escriba los siguientes valores.

    * **Suscripción**: seleccione una suscripción de Azure.
    * **Grupo de recursos**: seleccione **Crear nuevo**, escriba un nombre único para el grupo de recursos y, después, seleccione **Aceptar**.
    * **Región**: seleccione la ubicación del grupo de recursos. Ejemplo: **Centro de EE. UU.** .
    * **Acr Name** (Nombre de Acr): acepte el nombre que se genera para el registro o escriba otro nombre. Debe ser único globalmente.
    * **Acr Admin User Enabled** (Usuario administrador de ACR habilitado): acepte el valor predeterminado.
    * **Location** (Ubicación): acepte la ubicación que se genera para la réplica principal del registro, o bien especifique una ubicación como **Centro de EE. UU.** . 
    * **Acr Sku** (SKU de ACR): acepte el valor predeterminado.
    * **Acr Replica Location** (Ubicación de réplica de Acr): escriba una ubicación para la réplica del registro, use el nombre corto de la región. No debe coincidir con la ubicación del registro principal. Ejemplo: **westeurope**.

        :::image type="content" source="media/container-registry-get-started-geo-replication-template/template-properties.png" alt-text="Propiedades de la plantilla":::

1. Seleccione **Revisar y crear** y revise los términos y condiciones. Si está de acuerdo, seleccione **Crear**.

1. Cuando el registro se haya creado correctamente, recibirá una notificación:

     :::image type="content" source="media/container-registry-get-started-geo-replication-template/deployment-notification.png" alt-text="Notificación de Azure Portal":::

 Azure Portal se usa para implementar la plantilla. Además de Azure Portal, también puede usar Azure PowerShell, la CLI de Azure y API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../azure-resource-manager/templates/deploy-cli.md).

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Use Azure Portal o una herramienta como la CLI de Azure para examinar las propiedades del registro de contenedor.

1. En Azure Portal, busque Registros de contenedor y seleccione el registro de contenedor que creó.

1. En la página **Información general**, observe el valor del campo **Servidor de inicio de sesión** del registro. Use este identificador URI cuando utilice Docker para etiquetar e insertar imágenes en el registro. Para obtener información al respecto, consulte [Inserción de la primera imagen mediante la CLI de Docker](container-registry-get-started-docker-cli.md).

    :::image type="content" source="media/container-registry-get-started-geo-replication-template/registry-overview.png" alt-text="Información general del registro":::

1. En la página **Replicaciones**, confirme las ubicaciones de la réplica principal y de la réplica agregada a través de la plantilla. Si lo desea, agregue más réplicas en esta página.

    :::image type="content" source="media/container-registry-get-started-geo-replication-template/registry-replications.png" alt-text="Replicaciones del registro":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando no los necesite, elimine el grupo de recursos, el registro y la réplica del registro. Para ello, vaya a Azure Portal, seleccione el grupo de recursos que contiene el registro y, luego, seleccione **Eliminar grupo de recursos**.

Eliminación de un grupo de recursos

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha creado una instancia de Azure Container Registry con una plantilla de Resource Manager y ha configurado una réplica del registro en otra ubicación. Siga los tutoriales de Azure Container Registry para información más detallada de ACR.

> [!div class="nextstepaction"]
> [Tutoriales de Azure Container Registry](container-registry-tutorial-prepare-registry.md)

Para obtener un tutorial paso a paso que le guíe en el proceso de creación de una plantilla, consulte:

> [!div class="nextstepaction"]
> [Tutorial: Creación e implementación de su primera plantilla de Resource Manager](../azure-resource-manager/templates/template-tutorial-create-first-template.md)
