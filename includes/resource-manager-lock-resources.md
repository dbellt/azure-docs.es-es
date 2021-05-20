---
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 05/03/2021
ms.author: tomfitz
ms.openlocfilehash: 8864e8025c77fb0b4b6efc694d9f4938e9ed8ae7
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108737534"
---
1. En la hoja Configuración del recurso, del grupo de recursos o de la suscripción que desea bloquear, seleccione **Bloqueos**.

    :::image type="content" source="media/resource-manager-lock-resources/select-lock.png" alt-text="Seleccione el bloqueo.":::

1. Para agregar un bloqueo, seleccione **Agregar**. Si desea crear un bloqueo en un nivel primario, seleccione el elemento primario. El recurso seleccionado actualmente hereda el bloqueo del elemento primario. Por ejemplo, podría bloquear el grupo de recursos para aplicar un bloqueo a todos sus recursos.

    :::image type="content" source="media/resource-manager-lock-resources/add-lock.png" alt-text="Agregue el bloqueo.":::

1. Asigne al bloqueo un nombre y un nivel. Opcionalmente, puede agregar notas que describan el bloqueo.

    :::image type="content" source="media/resource-manager-lock-resources/set-lock.png" alt-text="Establezca el bloqueo.":::

1. Para eliminar el bloqueo, seleccione el botón **Eliminar**.

    :::image type="content" source="media/resource-manager-lock-resources/delete-lock.png" alt-text="Elimine el bloqueo.":::
