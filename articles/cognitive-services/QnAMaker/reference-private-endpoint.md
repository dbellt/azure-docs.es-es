---
title: 'Procedimiento para configurar un punto de conexión privado: QnA Maker'
description: Descripción de la creación de puntos de conexión privados disponible en QnA Maker administrado.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: reference
ms.date: 01/12/2021
ms.openlocfilehash: 00a55b852158abee01692d3c4ccbf1209c937cf4
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110379529"
---
# <a name="private-endpoints"></a>Puntos de conexión privados

Un punto de conexión privado de Azure es una interfaz de red que le conecta de forma privada y segura a un servicio con la tecnología de Azure Private Link. Ahora, la característica de respuesta a preguntas personalizada proporciona soporte para crear puntos de conexión privados para el servicio Azure Search.

Los puntos de conexión privados se proporcionan mediante [Azure Private Link](../../private-link/private-link-overview.md), como un servicio independiente. Para obtener más información sobre los costos, vea la [página de precios](https://azure.microsoft.com/pricing/details/private-link/). 

## <a name="prerequisites"></a>Requisitos previos
> [!div class="checklist"]
> * Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar.
> * Un [recurso de Text Analytics](https://ms.portal.azure.com/?quickstart=true#create/Microsoft.CognitiveServicesTextAnalytics) (con la característica de respuesta a preguntas personalizada) habilitado en Azure Portal. Recuerde el identificador de Azure Active Directory, la suscripción y el nombre del recurso de Text Analytics que seleccionó al crear el recurso.

## <a name="steps-to-enable-private-endpoint"></a>Pasos para habilitar el punto de conexión privado
1. Asigne el rol *Colaborador* al servicio Text Analytics en la instancia del servicio Azure Search. Para esta operación se necesita acceso de *Propietario* a la suscripción. Vaya a la pestaña Identidad del recurso del servicio para obtener la identidad.

> [!div class="mx-imgBorder"]
> ![Identidad de Text Analytics](../QnAMaker/media/qnamaker-reference-private-endpoints/private-endpoints-identity.png)

2. Agregue la identidad anterior como *Colaborador*; para ello, vaya a la pestaña IAM del servicio Azure Search.

![IAM de servicio administrado](../QnAMaker/media/qnamaker-reference-private-endpoints/private-endpoint-access-control.png)

3. Haga clic en *Agregar asignaciones de roles*, agregue la identidad y haga clic en *Guardar*.

![Asignación de roles administrados](../QnAMaker/media/qnamaker-reference-private-endpoints/private-endpoint-role-assignment.png)

4. Ahora, vaya a la pestaña *Redes* de la instancia del servicio Azure Search y cambie los datos de conectividad de punto de conexión de *Públicos* a *Privados*. Esta operación es un proceso de larga duración y puede tardar hasta 30 minutos en completarse. 

![Redes de Azure Search administradas](../QnAMaker/media/qnamaker-reference-private-endpoints/private-endpoint-networking.png)

5. Vaya a la pestaña *Redes* del servicio Text Analytics; en *Permitir acceso desde*, seleccione la opción *Redes y puntos de conexión privados seleccionados* y haga clic en *Guardar*.
 
> [!div class="mx-imgBorder"]
> ![Redes de Text Analytics](../QnAMaker/media/qnamaker-reference-private-endpoints/private-endpoint-networking-custom-qna.png)

Esto establecerá una conexión de punto de conexión privado entre el servicio Text Analytics y la instancia del servicio Azure Cognitive Search. Puede comprobar la conexión del punto de conexión privado en la pestaña *Redes* de la instancia del servicio Azure Cognitive Search. Una vez que se haya completado toda la operación, ya podrá usar el servicio Text Analytics. 

![Servicio de redes administrado](../QnAMaker/media/qnamaker-reference-private-endpoints/private-endpoint-networking-3.png)


## <a name="support-details"></a>Detalles sobre compatibilidad
 * No se admiten cambios en el servicio Azure Cognitive Search una vez que se habilita el acceso privado al servicio Text Analytics. Si cambia el servicio Azure Cognitive Search desde la pestaña "Características" después de haber habilitado el acceso privado, ya no se podrá utilizar el servicio Text Analytics.
 * Después de establecer la conexión de punto de conexión privado, si cambia las redes del servicio Azure Cognitive Search a "Públicas", no podrá usar el servicio Text Analytics. Para que la conexión del punto de conexión privado funcione, las redes del servicio Azure Search deben ser "Privadas".
