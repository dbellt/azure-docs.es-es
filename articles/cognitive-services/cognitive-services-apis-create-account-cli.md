---
title: Creación de un recurso de Cognitive Services con la CLI de Azure
titleSuffix: Azure Cognitive Services
description: Comience a usar Azure Cognitive Services mediante la creación de un recurso y la suscripción a él con la interfaz de la línea de comandos de Azure.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
keywords: cognitive services, inteligencia cognitiva, soluciones cognitivas, servicios de inteligencia artificial, servicios de IA
ms.topic: quickstart
ms.date: 3/22/2021
ms.author: aahi
ms.openlocfilehash: 26e3b264b7268f7a9ffdb592beef7d76844646f5
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107789148"
---
# <a name="quickstart-create-a-cognitive-services-resource-using-the-azure-command-line-interfacecli"></a>Inicio rápido: Creación de un recurso de Cognitive Services mediante la interfaz de la línea de comandos (CLI) de Azure

Use esta guía de inicio rápido para empezar a trabajar con Azure Cognitive Services mediante la [interfaz de la línea de comandos (CLI) de Azure](/cli/azure/install-azure-cli).

Azure Cognitive Services es un conjunto de servicios en la nube con API de REST y SDK de biblioteca cliente que ayudan a los desarrolladores a integrar la inteligencia cognitiva en las aplicaciones sin tener inteligencia artificial (IA) directa ni habilidades o conocimientos sobre ciencia de datos. Azure Cognitive Services permite a los desarrolladores agregar fácilmente características cognitivas en sus aplicaciones con soluciones cognitivas que pueden ver, oír, hablar, comprender e incluso empezar a pensar.

Los servicios de Cognitive Services se representan por medio de [recursos](../azure-resource-manager/management/manage-resources-portal.md) de Azure que se crean en la suscripción de Azure. Después de crear el recurso, use las claves y el punto de conexión generados para autenticar las aplicaciones.

En este artículo de inicio rápido, aprenderá a suscribirse a Azure Cognitive Services y a crear una cuenta con una suscripción de un solo servicio o de varios servicios mediante la [interfaz de la línea de comandos de Azure (CLI)](/cli/azure/install-azure-cli). Estos servicios se representan mediante los [recursos](../azure-resource-manager/management/manage-resources-portal.md) de Azure, que le permitirán conectarse a una o varias API de Azure Cognitive Services.

[!INCLUDE [cognitive-services-subscription-types](../../includes/cognitive-services-subscription-types.md)]

## <a name="prerequisites"></a>Prerrequisitos

* Una suscripción de Azure válida: [cree una de forma gratuita](https://azure.microsoft.com/free/cognitive-services).
* [Interfaz de la línea de comandos (CLI) de Azure](/cli/azure/install-azure-cli)

## <a name="install-the-azure-cli-and-sign-in"></a>Instale la CLI de Azure e inicie sesión.

Instale la [CLI de Azure](/cli/azure/install-azure-cli). Para iniciar sesión en la instalación local de la CLI, ejecute el comando [az login](/cli/azure/reference-index#az_login):

```azurecli-interactive
az login
```

También puede usar el botón verde **Probar** para ejecutar estos comandos en su navegador.

## <a name="create-a-new-azure-cognitive-services-resource-group"></a>Creación de un nuevo grupo de recursos de Azure Cognitive Services

Antes de crear un recurso de Cognitive Services, debe tener un grupo de recursos de Azure que contenga el recurso. Al crear un nuevo recurso, puede elegir entre crear un grupo de recursos o usar uno existente. En este artículo se muestra cómo crear un nuevo grupo de recursos.

### <a name="choose-your-resource-group-location"></a>Elija la ubicación de su grupo de recursos

Para crear un recurso, necesitará una de las ubicaciones de Azure disponibles para su suscripción. Puede recuperar una lista de ubicaciones disponibles con el comando [az account list-locations](/cli/azure/account#az_account_list_locations). Se puede acceder a la mayoría de las instancias de Cognitive Services desde varios lugares. Elija el más cercano a usted o consulte las ubicaciones que están disponibles para el servicio.

> [!IMPORTANT]
> * No olvide su ubicación de Azure, ya que la necesitará cuando llame a Azure Cognitive Services.
> * La disponibilidad de algunas instancias de Cognitive Services puede variar según la región. Para obtener más información, consulte [Productos de Azure disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services).

```azurecli-interactive
az account list-locations \
    --query "[].{Region:name}" \
    --out table
```

Una vez que tenga su ubicación de Azure, cree un grupo de recursos en la CLI de Azure mediante el comando [az group create](/cli/azure/group#az_group_create).

En el ejemplo que tiene a continuación, reemplace la ubicación de Azure `westus2` con una de las ubicaciones de Azure disponibles para su suscripción.

```azurecli-interactive
az group create \
    --name cognitive-services-resource-group \
    --location westus2
```

## <a name="create-a-cognitive-services-resource"></a>Creación de un recurso de Cognitive Services

### <a name="choose-a-cognitive-service-and-pricing-tier"></a>Elegir una instancia de Cognitive Services y el plan de tarifa

Al crear un nuevo recurso, deberá conocer el "tipo" de servicio que quiere usar, junto con el [plan de tarifa](https://azure.microsoft.com/pricing/details/cognitive-services/) (o sku) que quiere. Usará esta y otra información como parámetros al crear el recurso.

### <a name="multi-service"></a>Varios servicios

| Servicio                    | Clase                      |
|----------------------------|---------------------------|
| Varios servicios Consulte la página de [precios](https://azure.microsoft.com/pricing/details/cognitive-services/) para más información.            | `CognitiveServices`     |


> [!NOTE]
> Muchas de las instancias siguientes de Cognitive Services tienen un nivel gratuito que puede usar para probar el servicio. Para usar el nivel gratuito, use `F0` como sku del recurso.

### <a name="vision"></a>Visión

| Servicio                    | Clase                      |
|----------------------------|---------------------------|
| Computer Vision            | `ComputerVision`          |
| Custom Vision: Prediction | `CustomVision.Prediction` |
| Custom Vision: Training   | `CustomVision.Training`   |
| Caras                       | `Face`                    |
| Form Recognizer            | `FormRecognizer`          |
| Ink Recognizer             | `InkRecognizer`           |

### <a name="speech"></a>Voz

| Servicio            | Clase                 |
|--------------------|----------------------|
| Speech Services    | `SpeechServices`     |
| Reconocimiento de voz | `SpeakerRecognition` |

### <a name="language"></a>Idioma

| Servicio            | Clase                |
|--------------------|---------------------|
| Descripción del formulario | `FormUnderstanding` |
| LUIS               | `LUIS`              |
| QnA Maker          | `QnAMaker`          |
| Text Analytics     | `TextAnalytics`     |
| Traducción del texto   | `TextTranslation`   |

### <a name="decision"></a>Decisión

| Servicio           | Clase               |
|-------------------|--------------------|
| Anomaly Detector  | `AnomalyDetector`  |
| Content Moderator | `ContentModerator` |
| Personalizer      | `Personalizer`     |

Puede encontrar una lista de "tipos" de Cognitive Services disponibles con el comando [az cognitiveservices list-types](/cli/azure/cognitiveservices/account#az_cognitiveservices_account_list_kinds):

```azurecli-interactive
az cognitiveservices account list-kinds
```

### <a name="add-a-new-resource-to-your-resource-group"></a>Agregar un nuevo recurso al grupo de recursos

Para crear y suscribirse a un nuevo recurso de Cognitive Services, use el comando [az cognitiveservices account create](/cli/azure/cognitiveservices/account#az_cognitiveservices_account_create). Este comando agrega un nuevo recurso facturable al grupo de recursos creado anteriormente. Al crear este nuevo recurso, deberá conocer el "tipo" de servicio que quiere usar, junto con su plan de tarifa (o sku) y una ubicación de Azure:

Puede crear un recurso F0 (gratuito) para Anomaly Detector, denominado `anomaly-detector-resource` con el siguiente comando.

```azurecli-interactive
az cognitiveservices account create \
    --name anomaly-detector-resource \
    --resource-group cognitive-services-resource-group \
    --kind AnomalyDetector \
    --sku F0 \
    --location westus2 \
    --yes
```

[!INCLUDE [Register Azure resource for subscription](./includes/register-resource-subscription.md)]

## <a name="get-the-keys-for-your-resource"></a>Obtención de las claves del recurso

Para iniciar sesión en su instalación local de la interfaz de línea de comandos (CLI), use el comando [az login](/cli/azure/reference-index#az_login).

```azurecli-interactive
az login
```

Use el comando [az cognitiveservices account keys list](/cli/azure/cognitiveservices/account/keys#az_cognitiveservices_account_keys_list) para obtener las claves de su recurso de Cognitive Services.

```azurecli-interactive
    az cognitiveservices account keys list \
    --name anomaly-detector-resource \
    --resource-group cognitive-services-resource-group
```

[!INCLUDE [cognitive-services-environment-variables](../../includes/cognitive-services-environment-variables.md)]

## <a name="pricing-tiers-and-billing"></a>Planes de tarifas y facturación

Los planes de tarifas (y la cantidad que se factura) se basan en el número de transacciones que se envían con la información de autenticación. Cada plan de tarifa especifica lo siguiente:
* El número máximo de transacciones permitidas por segundo (TPS).
* las características de servicio habilitadas en el plan de tarifa.
* El costo de una cantidad predefinida de transacciones. Por encima de esta cantidad se producirá un cargo adicional, tal como se especifica en los [detalles de precios](https://azure.microsoft.com/pricing/details/cognitive-services/custom-vision-service/) del servicio.

## <a name="get-current-quota-usage-for-your-resource"></a>Obtener el uso de la cuota actual del recurso

Use el comando [az cognitiveservices account list-usage](/cli/azure/cognitiveservices/account#az_cognitiveservices_account_list_usage) para obtener el uso del recurso de Cognitive Services.

```azurecli-interactive
az cognitiveservices account list-usage \
    --name anomaly-detector-resource \
    --resource-group cognitive-services-resource-group \
    --subscription subscription-name
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar un recurso de Cognitive Services, puede eliminar el recurso o el grupo de recursos. Al eliminar el grupo de recursos también se eliminan los demás recursos incluidos en el grupo.

Para eliminar el grupo de recursos y sus recursos asociados, use el comando az group delete.

```azurecli-interactive
az group delete --name cognitive-services-resource-group
```

## <a name="see-also"></a>Consulte también

* Consulte **[Autenticación de solicitudes en Azure Cognitive Services](authentication.md)** para aprender a trabajar de forma segura con Cognitive Services.
* Consulte **[¿Qué es Azure Cognitive Services?](./what-are-cognitive-services.md)** para obtener una lista de las distintas categorías de Cognitive Services.
* Consulte **[Compatibilidad con idiomas naturales](language-support.md)** para ver la lista de los idiomas naturales admitidos por Cognitive Services.
* Consulte **[Contenedores de Azure Cognitive Services](cognitive-services-container-support.md)** para aprender a usar Cognitive Services en un entorno local.
* Consulte **[Planeamiento y administración de los costos de Azure Cognitive Services](plan-manage-costs.md)** para estimar el costo de usar Cognitive Services.
