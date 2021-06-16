---
title: 'Inicio rápido: Creación de una instancia de Azure API Management mediante una plantilla de Resource Manager'
description: Aprenda a crear una instancia de Azure API Management en el nivel de desarrollador mediante una plantilla de Azure Resource Manager.
services: azure-resource-manager
author: dlepow
ms.service: azure-resource-manager
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: danlep
ms.date: 10/09/2020
ms.openlocfilehash: a27d1240329802a4a1d7eecd117ef60ce06436fb
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111891489"
---
# <a name="quickstart-create-a-new-azure-api-management-service-instance-using-an-arm-template"></a>Inicio rápido: Creación de una instancia del servicio API Management mediante una plantilla de Resource Manager

En este inicio rápido se describe cómo usar una plantilla de Azure Resource Manager para crear una instancia del servicio Azure API Management (APIM). APIM ayuda a las organizaciones a publicar API para desarrolladores internos, externos y asociados, y así darles acceso a todas las posibilidades que ofrecen sus datos y servicios. API Management proporciona las competencias esenciales para garantizar un programa de API de éxito mediante compromisos con desarrolladores, información detallada empresarial, análisis, seguridad y protección. APIM le permite crear y administrar modernas puertas de enlace de API para los servicios back-end existentes hospedados en cualquier lugar. Para más información, consulte la [introducción](api-management-key-concepts.md).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.apimanagement%2Fazure-api-management-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/azure-api-management-create/).

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.apimanagement/azure-api-management-create/azuredeploy.json":::

El siguiente recurso se define en la plantilla:

- **[Microsoft.ApiManagement/service](/azure/templates/microsoft.apimanagement/service)**

Puede encontrar más ejemplos de plantillas de API Management en [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Apimanagement&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir una plantilla. La plantilla crea una instancia del servicio API Management con un nombre generado automáticamente.

    [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.apimanagement%2Fazure-api-management-create%2Fazuredeploy.json)

    En este ejemplo, la instancia se configura en el nivel de desarrollador, una opción económica para evaluar Azure API Management. Este nivel no puede utilizarse en producción. Para más información sobre el escalado de los niveles de API Management, consulte [Actualización y escalado](upgrade-and-scale.md).

1. Seleccione o escriba los siguientes valores.
    - **Suscripción**: seleccione una suscripción de Azure.
    - **Grupo de recursos**: seleccione **Crear nuevo**, escriba un nombre único para el grupo de recursos y, después, seleccione **Aceptar**.
    - **Región**: seleccione la ubicación del grupo de recursos. Ejemplo: **Centro de EE. UU.** .
    - **Publisher Email** (Dirección de correo del editor): escriba una dirección de correo electrónico donde recibir las notificaciones.
    - **Publisher Name** (Nombre del editor): escriba un nombre para el editor de la API.
    - **SKU**: acepte el valor predeterminado de **Developer** (Desarrollador).
    - **Sku Count** (Número de SKU): acepte el valor predeterminado.
    - **Location** (Ubicación): acepte la ubicación generada para el servicio API Management.

    :::image type="content" source="media/quickstart-arm-template/create-instance-template.png" alt-text="Propiedades de la plantilla de API Management":::

1. Seleccione **Revisar y crear** y revise los términos y condiciones. Si está de acuerdo, seleccione **Crear**.

    > [!TIP]
    >  Normalmente, la creación y activación de una instancia del servicio API Management en el nivel de desarrollador tarda entre 30 y 40 minutos.

1. Cuando la instancia se haya creado correctamente, recibirá una notificación:

    :::image type="content" source="media/quickstart-arm-template/deployment-notification.png" alt-text="Notificación de implementación":::

 Azure Portal se usa para implementar la plantilla. Además de Azure Portal, también puede usar Azure PowerShell, la CLI de Azure y API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../azure-resource-manager/templates/deploy-cli.md).

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Use Azure Portal para comprobar los recursos implementados, o utilice herramientas, como la CLI de Azure o Azure PowerShell, para enumerar los recursos implementados.

1. En [Azure Portal](https://portal.azure.com), busque y seleccione **Servicios API Management**, y seleccione la instancia de servicio que ha creado.
1. Revise las propiedades del servicio en la página de **información general**.

:::image type="content" source="media/quickstart-arm-template/service-instance-created.png" alt-text="Página de información general del servicio":::

Cuando la instancia de servicio de API Management está en línea, está listo para usarla. Comience con el tutorial [Importación y publicación de la primera API](import-and-publish.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si tiene previsto seguir trabajando con los tutoriales siguientes, podría conservar la instancia de API Management. Cuando ya no lo necesite, elimine el grupo de recursos, que elimina los recursos que contiene.

1. En [Azure Portal](https://portal.azure.com), busque la opción **Grupos de recursos** y selecciónela. También puede seleccionar **Grupos de recursos** en la página **principal**.
1. En la página **Grupos de recursos**, seleccione el grupo de recursos.
1. En la página del grupo de recursos, seleccione **Eliminar grupo de recursos**.

    Eliminación de un grupo de recursos
1. Escriba el nombre del grupo de recursos y, después, seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Importación y publicación de la primera API](import-and-publish.md)
