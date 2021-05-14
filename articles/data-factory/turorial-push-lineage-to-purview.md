---
title: Inserción de datos de linaje de Data Factory en Azure Purview
description: Información sobre cómo insertar datos de linaje de Data Factory en Azure Purview
ms.author: jingwang
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.custom:
- seo-lt-2019
- references_regions
ms.date: 12/3/2020
ms.openlocfilehash: c1b6f8856d7cff88abd59661d06422cdb0fbe271
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109481037"
---
# <a name="push-data-factory-lineage-data-to-azure-purview-preview"></a>Inserción de datos de linaje de Data Factory en Azure Purview (versión preliminar)

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este tutorial, usará la interfaz de usuario (UI) de Data Factory para crear una canalización que ejecute actividades y transmita datos de linaje a la cuenta de Azure Purview. Después, puede ver toda la información de linaje en su cuenta de Azure Purview.

## <a name="prerequisites"></a>Requisitos previos
* **Suscripción de Azure**. Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* **Azure Data Factory**. Si no tiene ninguna instancia de Azure Data Factory, consulte el artículo sobre la [creación de una instancia de Azure Data Factory](./quickstart-create-data-factory-portal.md).
* **Cuenta de Azure Purview**. La cuenta de Purview captura todos los datos de linaje que genera la factoría de datos. Si no dispone de una cuenta de Azure Purview, consulte [Creación de una instancia de Azure Purview](../purview/create-catalog-portal.md).


## <a name="run-data-factory-activities-and-push-lineage-data-to-azure-purview"></a>Ejecución de actividades de Data Factory e inserción de datos de linaje en Azure Purview
### <a name="step-1--connect-data-factory-to-your-purview-account"></a>Paso 1:  Conexión de Data Factory a la cuenta de Purview
Inicie sesión en su cuenta de Purview en el portal de Purview y vaya a **Centro de administración**. Elija **Data Factory** en **Conexiones externas** y haga clic en el botón **Nuevo** para crear una conexión a una nueva instancia de Data Factory. 
[![Captura de pantalla para crear una conexión entre Data Factory y la cuenta de Purview](./media/data-factory-purview/connect-adf-to-purview.png) ](./media/data-factory-purview/connect-adf-to-purview.png#lightbox)

En la página emergente, puede elegir la instancia de Data Factory que desea conectar a esta cuenta de Purview. 
![Captura de pantalla de una nueva conexión](./media/data-factory-purview/new-adf-purview-connection.png)

Puede comprobar el estado después de crear la conexión. **Conectado** significa que la conexión entre Data Factory y esta instancia de Purview se estableció correctamente. 
> [!NOTE]
> Debe tener asignado alguno de los roles siguientes en la cuenta de Purview y el rol **Colaborador** en Data Factory para crear la conexión entre Data Factory y Azure Purview.
> - Propietario
> - Administrador de acceso de usuario

### <a name="step-2-run-copy-and-dataflow-activities-in-data-factory"></a>Paso 2: Ejecución de actividades de copia y flujo de datos en Data Factory
Puede crear canalizaciones, actividades de copia y actividades de flujo de datos en Data Factory. No se requiere ninguna configuración adicional para la captura de datos de linaje. Los datos de linaje se capturarán automáticamente durante la ejecución de las actividades.
![Captura de pantalla de la actividad de copia y flujo de datos](./media/data-factory-purview/adf-activities-for-lineage.png) Si no sabe cómo crear actividades de copia y flujo de datos, consulte [Copia de datos desde Azure Blob Storage hasta una base de datos de Azure SQL Database mediante Azure Data Factory](./tutorial-copy-data-portal.md) y [Transformación de datos mediante flujos de datos de asignación](./tutorial-data-flow.md).

### <a name="step-3-run-execute-ssis-package-activities-in-data-factory"></a>Paso 3: Ejecución de actividades de ejecución de paquetes SSIS en Data Factory
Puede crear canalizaciones y actividades de ejecución de paquetes SSIS en Data Factory. No se requiere ninguna configuración adicional para la captura de datos de linaje. Los datos de linaje se capturarán automáticamente durante la ejecución de las actividades.
![Captura de pantalla de la actividad de ejecución de paquetes SSIS](./media/data-factory-purview/ssis-activities-for-lineage.png)

Si no sabe cómo crear actividades de ejecución de paquetes SSIS, consulte [Ejecución de paquetes SSIS en Azure](./tutorial-deploy-ssis-packages-azure.md).

### <a name="step-4-view-lineage-information-in-your-purview-account"></a>Paso 4: Visualización de la información de linaje en la cuenta de Purview
Vuelva a la cuenta de Purview. En la página principal, seleccione **Examinar recursos**. Elija el recurso que desee y haga clic en la pestaña Linaje. Verá toda la información de linaje.
[![Captura de pantalla de la cuenta de Purview](./media/data-factory-purview/view-dataset.png) ](./media/data-factory-purview/view-dataset.png#lightbox)

Puede ver los datos de linaje de la actividad de copia.
[![Captura de pantalla del linaje de copia](./media/data-factory-purview/copy-lineage.png) ](./media/data-factory-purview/copy-lineage.png#lightbox)

También puede ver los datos de linaje de la actividad de flujo de datos.
[![Captura de pantalla del linaje de flujo de datos](./media/data-factory-purview/dataflow-lineage.png) ](./media/data-factory-purview/dataflow-lineage.png#lightbox)

> [!NOTE] 
> Para el linaje de la actividad de flujo de datos, solo se admiten el origen y el receptor. Todavía no se admite el linaje de transformación del flujo de datos.

También puede ver los datos de linaje de la actividad de ejecución de paquetes SSIS.
[![Captura de pantalla del linaje de SSIS](./media/data-factory-purview/ssis-lineage.png) ](./media/data-factory-purview/ssis-lineage.png#lightbox)

> [!NOTE] 
> En cuanto al linaje de la actividad de ejecución de paquetes SSIS, solo se admiten el origen y el destino. Todavía no se admite el linaje de transformación.

## <a name="next-steps"></a>Pasos siguientes
[Guía de usuario del linaje de Data Catalog](../purview/catalog-lineage-user-guide.md)

[Conexión de Data Factory a Azure Purview](connect-data-factory-to-azure-purview.md)