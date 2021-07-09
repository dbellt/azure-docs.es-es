---
title: Creación de un recurso de Cognitive Services en Azure Portal
titleSuffix: Azure Cognitive Services
description: Comience a usar Azure Cognitive Services mediante la creación y suscripción a un recurso en Azure Portal.
services: cognitive-services
author: aahill
manager: nitinme
keywords: cognitive services, inteligencia cognitiva, soluciones cognitivas, servicios de inteligencia artificial, servicios de IA
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 06/04/2021
ms.author: aahi
ms.openlocfilehash: fb28a431f9f2f136d177a5d51398bc2427a7134d
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111540164"
---
# <a name="quickstart-create-a-cognitive-services-resource-using-the-azure-portal"></a>Inicio rápido: Creación de un recurso de Cognitive Services con Azure Portal

Use esta guía de inicio rápido para empezar a usar Azure Cognitive Services. Después de crear un recurso de Cognitive Services en Azure Portal, obtendrá un punto de conexión y una clave para autenticar las aplicaciones.

Azure Cognitive Services es un conjunto de servicios en la nube con API de REST y SDK de biblioteca cliente, que ayudan a los desarrolladores a integrar la inteligencia cognitiva en las aplicaciones sin tener inteligencia artificial (IA) directa ni habilidades o conocimientos sobre ciencia de datos. Azure Cognitive Services permite a los desarrolladores agregar fácilmente características cognitivas en sus aplicaciones con soluciones cognitivas que pueden ver, oír, hablar, comprender e incluso empezar a pensar.

[!INCLUDE [cognitive-services-subscription-types](../../includes/cognitive-services-subscription-types.md)]

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure válida: [cree una de manera gratuita](https://azure.microsoft.com/free/cognitive-services/).

## <a name="create-a-new-azure-cognitive-services-resource"></a>Creación de un nuevo recurso de Azure Cognitive Services

1. Crea un recurso.

### <a name="multi-service-resource"></a>[Recurso de varios servicios](#tab/multiservice)

El recurso multiservicio se denomina **Cognitive Services** en el portal. [Cree un recurso de Cognitive Services](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAllInOne).

En este momento, el recurso multiservicio permite el acceso a los siguientes servicios de Cognitive Services:

* **Visión**: Computer Vision, Custom Vision, Form Recognizer, Face
* **Voz**: Voz
* **Idioma**: Language Understanding (LUIS), Text Analytics, Translator
* **Decisión**: Content Moderator

### <a name="single-service-resource"></a>[Recurso de servicio único](#tab/singleservice)

Use los vínculos siguientes si desea crear un recurso para los servicios de Cognitive Services disponibles:

| Visión                      | Voz                  | Idioma                          | Decisión             |
|-----------------------------|-------------------------|-----------------------------------|----------------------|
| [Computer Vision](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision)         | [Servicios de Voz](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesSpeechServices)     | [Lector inmersivo](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesImmersiveReader)              | [Anomaly Detector](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector) | 
| [Servicio Custom Vision](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesCustomVision) |  | [Language Understanding (LUIS)](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesLUISAllInOne) | [Content Moderator](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesContentModerator) | 
| [Face](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFace)                    |                         | [QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker)                     | [Personalizer](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesPersonalizer)     |
| [Form Recognizer](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer)        |                         | [Text Analytics](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics)                |  [Metrics Advisor](https://go.microsoft.com/fwlink/?linkid=2142156)                    |
| | | [Traductor](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextTranslation) | |

---

2. En la página **Crear**, proporcione la siguiente información:
<!-- markdownlint-disable MD024 -->

### <a name="multi-service-resource"></a>[Recurso de varios servicios](#tab/multiservice)

|Detalles del proyecto| Descripción   |
|--|--|
| **Suscripción** | Seleccione una de las suscripciones de Azure disponibles. |
| **Grupos de recursos** | El grupo de recursos de Azure que contendrá su recurso de Cognitive Services. Puede crear un nuevo grupo o agregarlo a uno ya existente. |
| **Región** | Ubicación de la instancia de Cognitive Services. Las diferentes ubicaciones pueden crear latencias, pero no tienen ningún impacto en la disponibilidad del tiempo de ejecución del recurso. |
| **Nombre** | Nombre descriptivo para el recurso de Cognitive Services. Por ejemplo, *MyCognitiveServicesResource*. |
| **Plan de tarifa** | Costo de la cuenta de Cognitive Services, que depende del uso y de las opciones que elija. Para obtener más información, consulte los [detalles de los precios](https://azure.microsoft.com/pricing/details/cognitive-services/).

<!--![Multi-service resource creation screen](media/cognitive-services-apis-create-account/resource_create_screen-multi.png)-->
:::image type="content" source="media/cognitive-services-apis-create-account/resource_create_screen-multi.png" alt-text="Pantalla de creación de un recurso de varios servicios":::

Lea y acepte las condiciones, según corresponda, y después seleccione **Revisar y crear**.

### <a name="single-service-resource"></a>[Recurso de servicio único](#tab/singleservice)

|Detalles del proyecto| Descripción   |
|--|--|
| **Suscripción** | Seleccione una de las suscripciones de Azure disponibles. |
| **Grupos de recursos** | El grupo de recursos de Azure que contendrá su recurso de Cognitive Services. Puede crear un nuevo grupo o agregarlo a uno ya existente. |
| **Región** | Ubicación de la instancia de Cognitive Services. Las diferentes ubicaciones pueden crear latencias, pero no tienen ningún impacto en la disponibilidad del tiempo de ejecución del recurso. |
| **Nombre** | Nombre descriptivo para el recurso de Cognitive Services. Por ejemplo, *MyCognitiveServicesResource*. |
| **Plan de tarifa** | Costo de la cuenta de Cognitive Services, que depende del uso y de las opciones que elija. Para obtener más información, consulte los [detalles de los precios](https://azure.microsoft.com/pricing/details/cognitive-services/).

<!--![Single-service resource creation screen](media/cognitive-services-apis-create-account/resource_create_screen.png)-->
:::image type="content" source="media/cognitive-services-apis-create-account/resource_create_screen.png" alt-text="Pantalla de creación de recursos de un solo servicio":::

Seleccione **Next: Virtual Network**  (Siguiente: Red virtual) y elija el tipo de acceso de red que desea permitir para el recurso y, a continuación, seleccione **Revisar y crear**.

---

[!INCLUDE [Register Azure resource for subscription](./includes/register-resource-subscription.md)]

## <a name="get-the-keys-for-your-resource"></a>Obtención de las claves del recurso

1. Una vez que el recurso se haya implementado correctamente, en **Pasos siguientes**, haga clic en **Ir al recurso**.

    ![Búsqueda de Cognitive Services](media/cognitive-services-apis-create-account/resource-next-steps.png)

2. En el panel de inicio rápido que se abre, puede acceder a la clave y al punto de conexión.

    ![Obtención de una clave y un punto de conexión](media/cognitive-services-apis-create-account/get-cog-serv-keys.png)

[!INCLUDE [cognitive-services-environment-variables](../../includes/cognitive-services-environment-variables.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos también se eliminan los demás recursos incluidos en el grupo.

1. En Azure Portal, expanda el menú de la izquierda para abrir el menú de servicios y elija **Grupos de recursos** para ver una lista con sus grupos de recursos.
2. Busque el grupo de recursos que contiene el recurso que quiere eliminar.
3. Haga clic con el botón derecho en la lista de grupos de recursos. Seleccione **Eliminar grupo de recursos** y confirme.

Si necesita recuperar un recurso eliminado, consulte el artículo sobre la [recuperación de recursos de Cognitive Services eliminados](manage-resources.md).

## <a name="see-also"></a>Consulte también

* Consulte **[Autenticación de solicitudes en Azure Cognitive Services](authentication.md)** para aprender a trabajar de forma segura con Cognitive Services.
* Consulte **[¿Qué es Azure Cognitive Services?](./what-are-cognitive-services.md)** para obtener una lista de las distintas categorías de Cognitive Services.
* Consulte **[Compatibilidad con idiomas naturales](language-support.md)** para ver la lista de los idiomas naturales admitidos por Cognitive Services.
* Consulte **[Contenedores de Azure Cognitive Services](cognitive-services-container-support.md)** para aprender a usar Cognitive Services en un entorno local.
* Consulte **[Planeamiento y administración de los costos de Azure Cognitive Services](plan-manage-costs.md)** para estimar el costo de usar Cognitive Services.
