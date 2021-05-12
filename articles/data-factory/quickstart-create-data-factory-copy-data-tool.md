---
title: Copia de datos mediante la herramienta Copy Data de Azure
description: Cree una instancia de Azure Data Factory y, luego, use la herramienta Copy Data para copiar los datos de una ubicación de Azure Blob Storage a otra.
author: dearandyxu
ms.author: yexu
ms.service: data-factory
ms.topic: quickstart
ms.date: 11/09/2020
ms.openlocfilehash: 034f90adc46b541377f62ac66ea721a30ed48637
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108742056"
---
# <a name="quickstart-use-the-copy-data-tool-to-copy-data"></a>Inicio rápido: Uso de la herramienta Copy Data para copiar datos

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Versión actual](quickstart-create-data-factory-copy-data-tool.md)

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En esta guía de inicio rápido, usará Azure Portal para crear una factoría de datos. A continuación, usará la herramienta Copy Data para crear una canalización que copia datos de una carpeta de una instancia de Azure Blob Storage a otra carpeta. 

> [!NOTE]
> Si no está familiarizado con Azure Data Factory, consulte [Introduction to Azure Data Factory](introduction.md) antes de seguir los pasos de esta guía de inicio rápido. 

[!INCLUDE [data-factory-quickstart-prerequisites](includes/data-factory-quickstart-prerequisites.md)] 

## <a name="create-a-data-factory"></a>Crear una factoría de datos

1. Inicie el explorador web **Microsoft Edge** o **Google Chrome**. Actualmente, la interfaz de usuario de Data Factory solo se admite en los exploradores web Microsoft Edge y Google Chrome.
1. Vaya a [Azure Portal](https://portal.azure.com). 
1. En el menú de Azure Portal, seleccione **Crear un recurso** > **Integración** > **Data Factory**:

    ![Creación de nueva factoría de datos](./media/doc-common-process/new-azure-data-factory-menu.png)

1. En la página **Nueva factoría de datos**, escriba **ADFTutorialDataFactory** en **Nombre**. 
 
   El nombre de la instancia de Azure Data Factory debe ser *único de forma global*. Si ve el siguiente error, cambie el nombre de la factoría de datos (por ejemplo, **&lt;suNombre&gt;ADFTutorialDataFactory**) e intente crearlo de nuevo. Para conocer las reglas de nomenclatura de los artefactos de Data Factory, consulte el artículo [Azure Data Factory: reglas de nomenclatura](naming-rules.md).
  
   ![Mensaje de error cuando un nombre no está disponible](./media/doc-common-process/name-not-available-error.png)
1. En **Suscripción**, seleccione la suscripción de Azure donde desea crear la factoría de datos. 
1. Para **Grupo de recursos**, realice uno de los siguientes pasos:
     
   - Seleccione en primer lugar **Usar existente** y después un grupo de recursos de la lista. 
   - Seleccione **Crear nuevo** y escriba el nombre de un grupo de recursos.   
         
   Para obtener más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](../azure-resource-manager/management/overview.md).  
1. En **Versión**, seleccione **V2**.
1. En **Ubicación**, seleccione la ubicación de la factoría de datos.

   En la lista solo se muestran las ubicaciones que admite Data Factory y dónde se almacenarán los metadatos de Azure Data Factory. Los almacenes de datos asociados (como Azure Storage y Azure SQL Database) y los procesos (como Azure HDInsight) que usa Data Factory se pueden ejecutar en otras regiones.

1. Seleccione **Crear**.

1. Una vez completada la creación, verá la página **Data Factory**. Seleccione el icono **Author & Monitor** (Creación y supervisión) para iniciar la aplicación de interfaz de usuario de Azure Data Factory en una pestaña independiente.
   
:::image type="content" source="./media/doc-common-process/data-factory-home-page.png" alt-text="Página principal de Azure Data Factory, con el icono de creación y supervisión":::

## <a name="start-the-copy-data-tool"></a>Inicio de la herramienta Copy Data

1. En la página **Let's get started** (Comencemos), seleccione el icono **Copy Data** para iniciar la herramienta Copy Data. 

   ![Icono Copy Data](./media/doc-common-process/get-started-page.png)

1. En la página **Propiedades** de la herramienta Copiar datos, puede especificar un nombre para la canalización y su descripción, y luego seleccionar **Siguiente**. 

   ![Página de propiedades](./media/quickstart-create-data-factory-copy-data-tool/copy-data-tool-properties-page.png)
1. En la página **Almacén de datos de origen**, realice los pasos siguientes:

    a. Haga clic en **+ Crear nueva conexión** para agregar una conexión.

    b. Seleccione el tipo de servicio vinculado que desea crear para la conexión de origen. En este tutorial, se usará **Azure Blob Storage**. Selecciónelo en la galería y, luego, elija **Continuar**.
    
    ![Selección de blob](./media/quickstart-create-data-factory-copy-data-tool/select-blob-source.png)

    c. En la página **New Linked Service (Azure Blob Storage)** [Nuevo servicio vinculado (Azure Blob Storage)], especifique un nombre para el servicio vinculado. Seleccione su cuenta de almacenamiento en la lista **Nombre de la cuenta de Storage** y, después, seleccione **Crear**. 

    ![Configuración de la cuenta de Azure Blob Storage](./media/quickstart-create-data-factory-copy-data-tool/configure-blob-storage.png)

    d. Seleccione el servicio vinculado recién creado como origen y, a continuación, haga clic en **Siguiente**.


1. En la página **Choose the input file or folder** (Elegir el archivo o la carpeta de entrada), complete los siguientes pasos:

   a. Haga clic en **Browse** (Examinar) para ir a la carpeta **adftutorial/input**, seleccione el archivo **emp.txt** y, finalmente, haga clic en **Choose** (Elegir). 

   d. Seleccione la casilla **Binary copy** (Copia binaria) para copiar el archivo tal cual está, después, seleccione **Siguiente**. 

   ![Página de elección del archivo o la carpeta de entrada](./media/quickstart-create-data-factory-copy-data-tool/select-binary-copy.png)


1. En la página **Destination data store** (Almacén de datos de destino), seleccione el servicio vinculado a **Azure Blob Storage** que acaba de crear y, después, seleccione **Next** (Siguiente). 

1. En la página **Choose the output file or folder** (Elegir el archivo o la carpeta de salida), escriba **adftutorial/output** como ruta de acceso a la carpeta y, después, seleccione **Next** (Siguiente). 

   ![Página de elección del archivo o la carpeta de salida](./media/quickstart-create-data-factory-copy-data-tool/configure-sink-path.png) 

1. En la página **Configuración**, seleccione **Siguiente** para usar las configuraciones predeterminadas. 

1. En la página **Resumen**, revise todos los valores y seleccione **Siguiente**. 

1. En la página **Deployment complete** (Implementación finalizada), haga clic en **Monitor** (Supervisión) para supervisar la canalización que ha creado. 

    ![Página de implementación completada](./media/quickstart-create-data-factory-copy-data-tool/deployment-page.png)

1. La aplicación cambia a la pestaña **Monitor** (Supervisión). En esta pestaña verá el estado de la canalización. Seleccione **Refresh** (Actualizar) para actualizar la lista. Haga clic en el vínculo en **PIPELINE NAME** (NOMBRE DE CANALIZACIÓN) para ver los detalles de la ejecución de actividad o volver a ejecutar la canalización. 
   
    ![Actualización de canalización](./media/quickstart-create-data-factory-copy-data-tool/refresh-pipeline.png)

1. En la página Activity runs (Ejecuciones de actividad), seleccione el vínculo **Details** (Detalles) (icono de gafas) en la columna **ACTIVITY NAME** (NOMBRE DE ACTIVIDAD) para obtener más detalles sobre la operación de copia. Para más información sobre las propiedades, consulte [Introducción a la actividad de copia](copy-activity-overview.md). 

1. Para volver a la vista Pipeline Runs (Ejecuciones de canalización), seleccione el vínculo **ALL pipeline runs** (Todas las ejecuciones de canalizaciones) en el menú de la ruta de navegación. Para actualizar la vista, seleccione **Refresh** (Actualizar). 

1. Compruebe que se crea el archivo **emp.txt** en la carpeta **output** del contenedor **adftutorial**. Si no existe la carpeta de salida, el servicio Data Factory la crea automáticamente. 

1. Cambie a la pestaña **Creador** situada encima de la pestaña **Supervisar** del panel izquierdo para editar los servicios, conjuntos de datos y canalizaciones vinculados. Para saber cómo editarlos en la interfaz de usuario de Data Factory, consulte [Crear una factoría de datos mediante Azure Portal](quickstart-create-data-factory-portal.md).

    ![Pestaña de selección de autor](./media/quickstart-create-data-factory-copy-data-tool/select-author.png)

## <a name="next-steps"></a>Pasos siguientes
La canalización de este ejemplo copia los datos de una ubicación a otra en una instancia de Azure Blob Storage. Para más información sobre el uso de Data Factory en otros escenarios, consulte los siguientes [tutoriales](tutorial-copy-data-portal.md). 
