---
title: Inicio rápido para crear un clúster de Azure Stream Analytics
description: Obtenga información sobre cómo crear un clúster de Azure Stream Analytics.
ms.service: stream-analytics
author: sidramadoss
ms.author: sidram
ms.topic: quickstart
ms.custom: mvc
ms.date: 04/09/2021
ms.openlocfilehash: 7298862d29c02709afe737558421b4a8743ca5b0
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107305506"
---
# <a name="quickstart-create-a-dedicated-azure-stream-analytics-cluster-using-azure-portal"></a>Inicio rápido: Creación de un clúster de Azure Stream Analytics dedicado mediante Azure Portal

Use Azure Portal para crear un clúster de Azure Stream Analytics. Un [clúster de Stream Analytics](cluster-overview.md) es una implementación de un solo inquilino que se puede usar para casos de uso de streaming complejos y exigentes. Puede ejecutar varios trabajos de Stream Analytics en un clúster de Stream Analytics.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Finalización de [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure Portal](stream-analytics-quick-create-portal.md).

## <a name="create-a-stream-analytics-cluster"></a>Creación de un clúster de Stream Analytics

En esta sección, vamos a crear un recurso de clúster de Stream Analytics.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Seleccione **Crear un recurso**. En el cuadro de búsqueda *Buscar en Marketplace*, escriba y seleccione **Clúster de Stream Analytics**. A continuación, seleccione **Agregar**.

   :::image type="content" source="./media/create-cluster/search-result.png" alt-text="Resultado de la búsqueda del clúster de Stream Analytics.":::

1. En la página **Creación de un clúster de Stream Analytics**, escriba la configuración básica del nuevo clúster.

   |Parámetro|Value|Descripción |
   |---|---|---|
   |Suscripción|Nombre de suscripción|Seleccione la suscripción a Azure que quiera usar para este clúster de Stream Analytics. |
   |Grupo de recursos|Definición de un nombre de grupo de recursos|Seleccione un grupo de recursos o seleccione **Crear nuevo** y escriba un nombre único para el grupo de recursos nuevo. |
   |Cluster Name|Un nombre único|Escriba un nombre para identificar su clúster de Stream Analytics.|
   |Location|Región más cercana a los orígenes de datos y los receptores|Seleccione una ubicación geográfica en la que pueda hospedar su clúster de Stream Analytics. Use la ubicación más cercana a los orígenes de datos y los receptores para el análisis de baja latencia.|
   |Capacidad de la unidad de streaming|Entre 36 y 216 |Para determinar el tamaño del clúster, realice una estimación de cuántos trabajos de Stream Analytics tiene previsto ejecutar y el total de unidades de streaming que necesitará el trabajo. Puede empezar con 36 unidades de streaming y, luego, realizar un escalado o una reducción vertical según sea necesario.|

   ![Crear clúster](./media/create-cluster/create-cluster.png)

1. Seleccione **Revisar + crear**. Puede omitir las secciones **Etiquetas**.

1. Revise la configuración del clúster y, luego, seleccione **Crear**. La creación del clúster es una operación de larga duración que puede tardar aproximadamente 60 minutos en completarse. Espere hasta que la página del portal muestre **Se completó la implementación** . Mientras tanto, puede crear y desarrollar los [trabajos de Stream Analytics](stream-analytics-quick-create-portal.md#create-a-stream-analytics-job) que quiera ejecutar en este clúster si aún no lo ha hecho.

1. Seleccione **Ir al recurso** para ir a la página del clúster de Stream Analytics.

## <a name="delete-your-cluster"></a>Eliminación del clúster

Puede eliminar el clúster de Stream Analytics si no tiene previsto ejecutar trabajos de Stream Analytics en él. Para eliminar el clúster, siga los pasos que se indican en Azure Portal:

1. Vaya a **Trabajos de Stream Analytics** en **Configuración** y detenga todos los trabajos que hay en ejecución.

1. Vaya a **Información general** del clúster. Seleccione **Eliminar** y siga las instrucciones para eliminar el clúster.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, aprendió a crear un clúster de Azure Stream Analytics. Avance al siguiente artículo para obtener información sobre cómo ejecutar un trabajo de Stream Analytics en el clúster:

> [!div class="nextstepaction"]
> [Administración de trabajos de Stream Analytics en un clúster de Stream Analytics](manage-jobs-cluster.md)
