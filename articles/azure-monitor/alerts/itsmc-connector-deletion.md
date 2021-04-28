---
title: Eliminación de conectores ITSM sin usar
description: En este artículo se explica cómo eliminar un conector ITSM y los grupos de acciones asociadas a él.
ms.topic: conceptual
author: nolavime
ms.author: v-jysur
ms.date: 12/29/2020
ms.custom: references_regions
ms.openlocfilehash: 3dc84ea6def9b762527226dbeb3e2eaab78ec200
ms.sourcegitcommit: aa00fecfa3ad1c26ab6f5502163a3246cfb99ec3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107387945"
---
# <a name="delete-unused-itsm-connectors"></a>Eliminación de conectores ITSM sin usar

El proceso de eliminación de conectores de administración de servicios de TI (ITSM) no usados tiene dos fases. Elimine todas las acciones asociadas a un conector ITSM y, a continuación, elimine el propio conector. Las acciones se eliminan primero porque las acciones sin un conector pueden provocar errores en la suscripción.

## <a name="delete-associated-actions"></a>Eliminación de acciones asociadas

1. En Azure Portal, seleccione **Monitor**.
  
    ![Captura de pantalla de la selección de Monitor.](media/itsmc-connector-deletion/itsmc-monitor-selection.png)

2. Seleccione **Alertas**.
   
    ![Captura de pantalla de la selección de Alertas.](media/itsmc-connector-deletion/itsmc-alert-selection.png)

3. Seleccione **Administrar acciones**.
   
    ![Captura de pantalla de la selección de Administrar acciones.](media/itsmc-connector-deletion/itsmc-actions-selection.png)

4. Seleccione un grupo de acciones asociado al conector ITSM que desea eliminar. En este artículo se usa un conector de Cherwell como ejemplo.
   
    ![Captura de pantalla de las acciones asociadas al conector de Cherwell.](media/itsmc-connector-deletion/itsmc-actions-screen.png)

5. Revise la información y, a continuación, seleccione **Eliminar grupo de acciones**.

    ![Captura de pantalla de la información del grupo de acciones y el botón para eliminar el grupo.](media/itsmc-connector-deletion/itsmc-action-deletion.png)

## <a name="delete-the-connector"></a>Eliminación del conector

1. En la barra de búsqueda, busque **servicedesk**. A continuación, seleccione **ServiceDesk** en la lista de recursos.

    ![Captura de pantalla de búsqueda y selección de ServiceDesk.](media/itsmc-connector-deletion/itsmc-connector-selection.png)

2. Seleccione **Conexiones de ITSM** y, a continuación, seleccione el conector de Cherwell.

    ![Captura de pantalla del conector I T S M de Cherwell.](media/itsmc-connector-deletion/itsmc-cherwell-connector.png)

3. Seleccione **Eliminar**.

    ![Captura de pantalla del botón Eliminar para el conector I T S M.](media/itsmc-connector-deletion/itsmc-connector-deletion.png)

## <a name="next-steps"></a>Pasos siguientes

* [Solución de problemas de un conector ITSM](./itsmc-resync-servicenow.md).
