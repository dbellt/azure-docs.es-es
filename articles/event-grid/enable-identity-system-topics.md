---
title: Habilitación de la identidad administrada en el tema del sistema de Azure Event Grid
description: En este artículo se describe cómo habilitar la identidad de servicio administrada para un tema del sistema de Azure Event Grid.
ms.topic: how-to
ms.date: 03/25/2021
ms.openlocfilehash: d8219cf9cba4ce0a4fb5ddbcf695f1ec72de36fe
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106280520"
---
# <a name="assign-a-system-managed-identity-to-an-event-grid-system-topic"></a>Asignación de una identidad administrada por el sistema a un tema del sistema de Event Grid
En este artículo, aprenderá a habilitar la identidad administrada por el sistema para un tema del sistema de Event Grid existente. Para más información sobre las identidades administradas, consulte [¿Qué son las identidades administradas de recursos de Azure?](../active-directory/managed-identities-azure-resources/overview.md).  

> [!IMPORTANT]
> Actualmente, no se puede habilitar una identidad administrada por el sistema al crear un nuevo tema del sistema, es decir, al crear una suscripción de eventos en un recurso de Azure que admite temas del sistema. 


## <a name="use-azure-portal"></a>Usar Azure Portal
El siguiente procedimiento muestra cómo habilitar una identidad administrada por el sistema para un tema del sistema. 

1. Vaya a [Azure Portal](https://portal.azure.com).
2. En la barra de búsqueda de la parte superior, busque **Temas del sistema de Event Grid**.
3. Seleccione el **tema del sistema** para el que quiere habilitar la identidad administrada. 
4. Seleccione **Identidad** en el menú de la izquierda. Esta opción no es visible para un tema del sistema que se encuentra en la ubicación global. 
5. **Active** el conmutador para habilitar la identidad. 
1. Seleccione **Guardar** en la barra de herramientas para guardar la configuración. 

    :::image type="content" source="./media/managed-service-identity/identity-existing-system-topic.png" alt-text="Página de identidad para un tema del sistema"::: 
1. Seleccione **Sí** en el mensaje de confirmación. 

    :::image type="content" source="./media/managed-service-identity/identity-existing-system-topic-confirmation.png" alt-text="Asignación de una identidad a un tema del sistema: confirmación"::: 
1. Confirme que ve el id. de objeto de la identidad administrada asignada por el sistema y un vínculo para asignar roles. 

    :::image type="content" source="./media/managed-service-identity/identity-existing-system-topic-completed.png" alt-text="Asignación de una identidad a un tema del sistema: completado"::: 

## <a name="global-azure-sources"></a>Orígenes globales de Azure
Solo puede habilitar la identidad administrada por el sistema para los recursos regionales de Azure. No se puede habilitar para los temas del sistema asociados a recursos globales de Azure, como suscripciones de Azure, grupos de recursos o Azure Maps. Los temas del sistema para estos orígenes globales tampoco están asociados a una región específica. La página **Identidad** del tema del sistema cuya ubicación está establecida en **Global** no se ve. 

:::image type="content" source="./media/managed-service-identity/system-topic-location-global.png" alt-text="Tema del sistema con la ubicación establecida en Global"::: 



## <a name="next-steps"></a>Pasos siguientes
Agregue la identidad a un rol adecuado (por ejemplo, Remitente de los datos de Service Bus) en el destino (por ejemplo, una cola de Service Bus). Para los pasos detallados, consulte [Concesión a la identidad administrada el acceso al destino de Event Grid](add-identity-roles.md). 