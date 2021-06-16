---
title: 'Configuración de registros de diagnóstico: Azure Event Hub | Microsoft Docs'
description: Obtenga información sobre cómo configurar registros de actividad y registros de diagnóstico para centros de eventos en Azure.
ms.topic: article
ms.date: 06/13/2021
ms.openlocfilehash: d981f1d01579719ecd055307f8a9877be94072d9
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112060818"
---
# <a name="set-up-diagnostic-logs-for-an-azure-event-hub"></a>Configuración de registros de diagnóstico de Azure Event Hubs

Puede ver dos tipos de registros para Event Hubs de Azure:

* **[Registros de actividad](../azure-monitor/essentials/platform-logs-overview.md)** : Estos registros contienen información sobre las operaciones realizadas en un trabajo. Los registros están siempre habilitados. Puede ver las entradas del registro de actividad si selecciona **Registro de actividad** en el panel izquierdo del espacio de nombres del centro de eventos en Azure Portal. Por ejemplo: "Crear o actualizar el espacio de nombres", "Crear o actualizar el centro de eventos".

    ![Registro de actividad de un espacio de nombres de Event Hubs](./media/event-hubs-diagnostic-logs/activity-log.png)
* **[Registros de diagnóstico](../azure-monitor/essentials/platform-logs-overview.md)** : los registros de diagnóstico proporcionan información más completa sobre las operaciones y acciones realizadas en el espacio de nombres mediante la API o mediante clientes de administración en el SDK de lenguaje. 
    
    En la sección siguiente se muestra cómo habilitar los registros de diagnóstico para un espacio de nombres de Event Hubs.

## <a name="enable-diagnostic-logs"></a>Habilitar registros de diagnóstico
Los registros de diagnóstico están inhabilitados de forma predeterminada. Para habilitarlos, siga estos pasos:

1.  En [Azure Portal](https://portal.azure.com), vaya al espacio de nombres de Event Hubs. 
2. Seleccione **Configuración de diagnóstico** en **Supervisión** en el panel izquierdo y, luego, elija **+ Agregar configuración de diagnóstico**. 

    ![Página Configuración de diagnóstico: Agregar configuración de diagnóstico](./media/event-hubs-diagnostic-logs/diagnostic-settings-page.png)
4. En la sección **Detalles de la categoría**, seleccione los **tipos de registros de diagnóstico** que quiere habilitar. Más adelante en este artículo encontrará información sobre estas categorías. 
5. En la sección **Detalles de destino**, establezca el destino de archivo que prefiera; por ejemplo, una cuenta de almacenamiento, un centro de eventos o un área de trabajo de Log Analytics.

    ![Agregar página de configuración de diagnóstico](./media/event-hubs-diagnostic-logs/aDD-diagnostic-settings-page.png)
6.  Seleccione **Guardar** en la barra de herramientas para guardar la configuración de diagnóstico.

    La nueva configuración surte efecto en unos 10 minutos. Después, los registros aparecen en el destino de archivo configurado, en el panel **Registros de diagnóstico**.

    Para obtener más información sobre el diagnóstico de configuraciones, consulte la [información general sobre los registros de diagnóstico de Azure](../azure-monitor/essentials/platform-logs-overview.md).

## <a name="diagnostic-logs-categories"></a>Categorías de registros de diagnósticos
[!INCLUDE [event-hubs-diagnostic-log-schema](../../includes/event-hubs-diagnostic-log-schema.md)]



## <a name="next-steps"></a>Pasos siguientes
- [Introducción a Event Hubs](./event-hubs-about.md)
- [Ejemplos de Event Hubs](sdks.md)
- Introducción a Event Hubs
    - [.NET Core](event-hubs-dotnet-standard-getstarted-send.md)
    - [Java](event-hubs-java-get-started-send.md)
    - [Python](event-hubs-python-get-started-send.md)
    - [JavaScript](event-hubs-node-get-started-send.md)
