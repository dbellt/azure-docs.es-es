---
title: Creación de un recurso de Azure Cognitive Services mediante plantillas de Resource Manager | Microsoft Docs
description: Cree un recurso de Azure Cognitive Service mediante una plantilla de Resource Manager.
keywords: cognitive services, cognitive solutions, cognitive intelligence, cognitive artificial intelligence
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: quickstart
ms.date: 06/04/2021
ms.author: aahi
ms.custom: subject-armqs
ms.openlocfilehash: b9fcd3d5eac8ddda890201bf5608e6e412f5d826
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112004320"
---
# <a name="quickstart-create-a-cognitive-services-resource-using-an-arm-template"></a>Inicio rápido: Creación de un recurso de Cognitive Services mediante una plantilla de Resource Manager

En este inicio rápido se describe cómo usar una plantilla de Azure Resource Manager para crear una instancia de Cognitive Services.

Azure Cognitive Services es un conjunto de servicios en la nube con API de REST y SDK de biblioteca cliente que ayudan a los desarrolladores a integrar la inteligencia cognitiva en las aplicaciones sin tener inteligencia artificial (IA) directa ni habilidades o conocimientos sobre ciencia de datos. Azure Cognitive Services permite a los desarrolladores agregar fácilmente características cognitivas en sus aplicaciones con soluciones cognitivas que pueden ver, oír, hablar, comprender e incluso empezar a pensar.

Cree un recurso mediante una plantilla de Azure Resource Manager. Este recurso de varios servicios le permite:

* Acceda a varias instancias de Azure Cognitive Services con una sola clave y un punto de conexión.
* Consolide la facturación de los servicios que usa.
* [!INCLUDE [terms-azure-portal](./includes/quickstarts/terms-azure-portal.md)]

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación de Cognitive Service en Azure](../media/template-deployments/deploy-to-azure.svg "Implementación de Cognitive Service en Azure")](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.cognitiveservices%2Fcognitive-services-universalkey%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

* Si no tiene una suscripción a Azure, [créela de forma gratuita](https://azure.microsoft.com/free/cognitive-services).

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/cognitive-services-universalkey/).

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.cognitiveservices/cognitive-services-universalkey/azuredeploy.json":::

En la plantilla, se define un recurso de Azure:
* [Microsoft.CognitiveServices/accounts](/azure/templates/microsoft.cognitiveservices/accounts): crea un recurso de Cognitive Services.

## <a name="deploy-the-template"></a>Implementación de la plantilla

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

1. Haga clic en botón **Deploy to Azure** (Implementar en Azure).

    [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.cognitiveservices%2Fcognitive-services-universalkey%2Fazuredeploy.json)

2. Escriba los siguientes valores:

    |Value  |Descripción  |
    |---------|---------|
    | **Suscripción** | Seleccione una suscripción de Azure. |
    | **Grupos de recursos** | Seleccione **Crear nuevo**, escriba un nombre único para el grupo de recursos y haga clic en **Aceptar**. |
    | **Región** | Seleccione una región.  Por ejemplo **Este de EE. UU.** |
    | **Nombre de Cognitive Service** | Reemplácelo por un nombre único para el recurso. Necesitará el nombre en la sección siguiente cuando valide la implementación. |
    | **Ubicación** | Reemplácelo por la región utilizada anteriormente. |
    | **Sku** | El [plan de tarifa](https://azure.microsoft.com/pricing/details/cognitive-services/) del recurso. |

    :::image type="content" source="media/arm-template/universal-key-portal-template.png" alt-text="Pantalla de creación de recursos":::.

3. Seleccione **Revisar y crear** y, a continuación, **Crear**. Una vez que el recurso ha finalizado correctamente la implementación, se resaltará el botón **Ir al recurso**.

# <a name="azure-cli"></a>[CLI de Azure](#tab/CLI)

> [!NOTE]
> La creación de `az deployment group` requiere la versión 2.6 de la CLI de Azure, o cualquier versión posterior. Para mostrar el tipo de versión `az --version`. Para más información, consulte la [documentación](/cli/azure/deployment/group).

Ejecute el siguiente script mediante la interfaz de la línea de comandos (CLI) de Azure [en la máquina local](/cli/azure/install-azure-cli) o en un explorador con el botón **Pruébelo**. Escriba un nombre y una ubicación (por ejemplo `centralus`) para un nuevo grupo de recursos y la plantilla de Resource Manager se usará para implementar un recurso de Cognitive Services dentro de él. Recuerde el nombre que use. Lo usará más adelante para validar la implementación.


```azurecli-interactive
read -p "Enter a name for your new resource group:" resourceGroupName &&
read -p "Enter the location (i.e. centralus):" location &&
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.cognitiveservices/cognitive-services-universalkey/azuredeploy.json" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri &&
echo "Press [ENTER] to continue ..." &&
read
```

---

[!INCLUDE [Register Azure resource for subscription](./includes/register-resource-subscription.md)]

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

# <a name="portal"></a>[Portal](#tab/portal)

Cuando finalice la implementación, podrá hacer clic en el botón **Ir al recurso** para ver el nuevo recurso. Para buscar el grupo de recursos:

1. Seleccione **Grupos de recursos** en el menú de navegación izquierdo.
2. Seleccione el nombre del grupo de recursos.

# <a name="azure-cli"></a>[CLI de Azure](#tab/CLI)

Mediante la CLI de Azure, ejecute el siguiente script y escriba el nombre del grupo de recursos que creó anteriormente.

```azurecli-interactive
echo "Enter the resource group where the Cognitive Services resource exists:" &&
read resourceGroupName &&
az cognitiveservices account list -g $resourceGroupName
```

---


## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos también se eliminan los demás recursos incluidos en el grupo.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

1. En Azure Portal, expanda el menú de la izquierda para abrir el menú de servicios y elija **Grupos de recursos** para ver una lista con sus grupos de recursos.
2. Busque el grupo de recursos que contiene el recurso que quiere eliminar.
3. Haga clic con el botón derecho en la lista de grupos de recursos. Seleccione **Eliminar grupo de recursos** y confirme.

# <a name="azure-cli"></a>[CLI de Azure](#tab/CLI)

Mediante la CLI de Azure, ejecute el siguiente script y escriba el nombre del grupo de recursos que creó anteriormente.

```azurecli-interactive
echo "Enter the resource group name, for deletion:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName
```

---

Si necesita recuperar un recurso eliminado, consulte el artículo sobre la [recuperación de recursos de Cognitive Services eliminados](manage-resources.md).

## <a name="see-also"></a>Consulte también

* Consulte **[Autenticación de solicitudes en Azure Cognitive Services](authentication.md)** para aprender a trabajar de forma segura con Cognitive Services.
* Consulte **[¿Qué es Azure Cognitive Services?](./what-are-cognitive-services.md)** para obtener una lista de las distintas categorías de Cognitive Services.
* Consulte **[Compatibilidad con idiomas naturales](language-support.md)** para ver la lista de los idiomas naturales admitidos por Cognitive Services.
* Consulte **[Contenedores de Azure Cognitive Services](cognitive-services-container-support.md)** para aprender a usar Cognitive Services en un entorno local.
* Consulte **[Planeamiento y administración de los costos de Azure Cognitive Services](plan-manage-costs.md)** para estimar el costo de usar Cognitive Services.
