---
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: include
ms.date: 05/03/2021
ms.openlocfilehash: 7b35a3c4b26640b872b161bd8130db728645f1d0
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108748294"
---
Una [conexión de servicio](/azure/devops/pipelines/library/service-endpoints) concede acceso a los recursos en la suscripción de Azure desde el proyecto de Azure DevOps.

1. En Azure DevOps, vaya al proyecto que contiene la canalización de destino. En la esquina inferior izquierda, seleccione **Configuración del proyecto**.
1. En **Canalizaciones**, seleccione **Conexiones de servicio**. En la esquina superior derecha, seleccione **Nueva conexión de servicio**.
1. En **Nueva conexión de servicio**, seleccione **Azure Resource Manager**.

    :::image type="content" source="./media/azure-app-configuration-service-connection/new-service-connection.png" alt-text="Captura de pantalla que muestra la selección de Azure Resource Manager en la lista desplegable Nueva conexión de servicio.":::
1. En el cuadro de diálogo **Método de autenticación**, seleccione **Entidad de servicio (automática)** para crear una nueva entidad de servicio, o seleccione **Entidad de servicio (manual)** para [usar una entidad de servicio existente](/azure/devops/pipelines/library/connect-to-azure?view=azure-devops#use-spn&preserve-view=true).
1. Escriba la suscripción y el recurso y un nombre para la conexión de servicio.

Si ha creado una nueva entidad de servicio, busque el nombre de la entidad de servicio asignada a la conexión del servicio. En el paso siguiente, agregará una nueva asignación de roles a esta entidad de servicio.

1. Vaya a **Configuración del proyecto** > **Conexiones de servicio**.
1. Seleccione la nueva conexión de servicio.
1. Seleccione **Administrar entidad de servicio**.
1. Anote el valor de **Nombre para mostrar**.

    :::image type="content" source="./media/azure-app-configuration-service-connection/service-principal-display-name.png" alt-text="Captura de pantalla que muestra el nombre para mostrar de la entidad de servicio.":::
