---
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: include
ms.date: 05/03/2021
ms.openlocfilehash: 7f5f062e35aad6b7623f1a2f97ab3b9133266ac6
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108748289"
---
Asigne las asignaciones de roles de App Configuration adecuadas a las credenciales que se van a usar en la tarea para que esta pueda acceder al almacén de App Configuration.

1. Vaya al almacén de App Configuration de destino. 
1. En el menú izquierdo, seleccione **Control de acceso (IAM)** .
1. En el panel derecho, seleccione **Agregar asignaciones de roles**.
    
    :::image type="content" source="./media/azure-app-configuration-role-assignment/add-role-assignment-button.png" alt-text="Captura de pantalla que muestra el botón Agregar asignaciones de roles.":::

1. En **Rol**, seleccione **Propietario de los datos de App Configuration**. Este rol permite que la tarea lea y escriba en el almacén de App Configuration. 
1. Seleccione la entidad de servicio asociada con la conexión de servicio que creó en la sección anterior.

    :::image type="content" source="./media/azure-app-configuration-role-assignment/add-role-assignment.png" alt-text="Captura de pantalla que muestra el cuadro de diálogo Agregar asignaciones de roles.":::
