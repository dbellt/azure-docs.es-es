---
author: vladvino
ms.service: api-management
ms.topic: include
ms.date: 04/16/2021
ms.author: vlvinogr
ms.openlocfilehash: 890cb1dc8d8a59f6b567497b968b53bb89ec1ecc
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108230729"
---
## <a name="append-other-apis"></a>Anexión de otras API

Puede crear una API a partir de las API expuestas por distintos servicios, como los siguientes:
* La especificación de OpenAPI
* Una API SOAP
* Una aplicación web hospedada en Azure App Service
* Aplicación de función de Azure
* Azure Logic Apps
* Azure Service Fabric

Para anexar una API distinta a la API existente, siga estos pasos. 

>[!NOTE] 
> Al importar otra API, las operaciones se anexan a la API actual.

1. Vaya a la instancia de Azure API Management en Azure Portal.

    :::image type="content" source="./media/api-management-append-apis/service-page.png" alt-text="Vaya a la instancia de Azure API Management.":::

1. Seleccione **API** en la página **Información general** o en el menú de la izquierda.

    :::image type="content" source="./media/api-management-append-apis/api-select.png" alt-text="Selección de las API":::

1. Haga clic en **...** junto a la API a la que desea anexar otra API.
1. Seleccione **Importar** en el menú desplegable.

    :::image type="content" source="./media/api-management-append-apis/append-01.png" alt-text="Selección de Importar":::

1. Seleccione un servicio desde el que se va a importar una API.

    :::image type="content" source="./media/api-management-append-apis/select-to-import.png" alt-text="Seleccionar servicio":::
