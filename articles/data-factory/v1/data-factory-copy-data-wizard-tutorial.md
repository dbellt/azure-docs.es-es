---
title: 'Tutorial: creación de una canalización con el Asistente para copia '
description: En este tutorial, creará una canalización de Azure Data Factory con una actividad de copia mediante el Asistente para copia compatible con Data Factory.
author: linda33wj
ms.service: data-factory
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 16618276abc5e7933e5a434fd8537b09ad104d7b
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108753414"
---
# <a name="tutorial-create-a-pipeline-with-copy-activity-using-data-factory-copy-wizard"></a>Tutorial: Creación de una canalización con la actividad de copia mediante el Asistente para copia de Data Factory
> [!div class="op_single_selector"]
> * [Introducción y requisitos previos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Asistente para copia](data-factory-copy-data-wizard-tutorial.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Plantilla de Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [API de .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte el [tutorial de la actividad de copia](../quickstart-create-data-factory-dot-net.md). 


En este tutorial se muestra cómo usar el **Asistente para copia** para copiar datos desde Azure Blob Storage hasta Azure SQL Database. 

El **Asistente para copia** de Azure Data Factory le permite crear rápidamente una canalización de datos que copia datos desde un almacén de datos de origen compatible a un almacén de datos de destino compatible. Por lo tanto, se recomienda utilizar el asistente como primer paso para crear una canalización de ejemplo para el escenario de movimiento de datos. Para obtener una lista de almacenes de datos admitidos como orígenes y destinos, consulte los [almacenes de datos admitidos](data-factory-data-movement-activities.md#supported-data-stores-and-formats).  

En este tutorial se muestra cómo crear una instancia de Azure Data Factory, iniciar el Asistente para copia, y realizar una serie de pasos en los que se proporcionan detalles acerca del escenario de ingesta y movimiento de datos. Al terminar los pasos del asistente, se crea automáticamente una canalización con una actividad de copia para copiar datos de Azure Blob Storage a Azure SQL Database. Para más información acerca de la actividad de copia, consulte las [actividades de movimiento de datos](data-factory-data-movement-activities.md).

## <a name="prerequisites"></a>Requisitos previos
Antes de realizar este tutorial, complete los requisitos previos que se enumeran en el artículo [Copia de datos de Almacenamiento de blobs en Base de datos SQL mediante Data Factory](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) .

## <a name="create-data-factory"></a>Creación de Data Factory
En este paso, utilizará el Portal de Azure para crear una factoría de datos de Azure llamada **ADFTutorialDataFactory**.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Haga clic en **Crear un recurso** en la esquina superior izquierda, después en **Datos y análisis** y en **Data Factory**. 
   
   ![New->DataFactory](./media/data-factory-copy-data-wizard-tutorial/new-data-factory-menu.png)
2. En la hoja **Nueva factoría de datos** :
   
   1. Escriba **ADFTutorialDataFactory** como **nombre**.
       El nombre de la instancia de Azure Data Factory debe ser único de forma global. Si recibe el error: `Data factory name "ADFTutorialDataFactory" is not available`, cambie el nombre de la factoría de datos (por ejemplo, yournameADFTutorialDataFactoryYYYYMMDD) e intente crearla de nuevo. Consulte el tema [Factoría de datos: reglas de nomenclatura](data-factory-naming-rules.md) para las reglas de nomenclatura para los artefactos de Factoría de datos.  
      
       ![Nombre de Factoría de datos no disponible](./media/data-factory-copy-data-wizard-tutorial/getstarted-data-factory-not-available.png)    
   2. Selección la **suscripción** de Azure.
   3. Para el grupo de recursos, realice uno de los siguientes pasos: 
      
      - Seleccione en primer lugar **Usar existente** y después un grupo de recursos existente.
      - Seleccione **Crear nuevo** y escriba un nombre para un grupo de recursos.
          
        En algunos de los pasos de este tutorial se supone que usa el nombre: **ADFTutorialResourceGroup** para el grupo de recursos. Para obtener más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](../../azure-resource-manager/management/overview.md).
   4. Seleccione una **ubicación** para la factoría de datos.
   5. Seleccione la casilla **Anclar al panel** en la parte inferior de la hoja.  
   6. Haga clic en **Crear**.
      
       ![Hoja Nueva Factoría de datos](media/data-factory-copy-data-wizard-tutorial/new-data-factory-blade.png)            
3. Una vez completada la creación, puede ver la hoja **Data Factory** como se muestra en la siguiente imagen:
   
   ![Página principal Factoría de datos](./media/data-factory-copy-data-wizard-tutorial/getstarted-data-factory-home-page.png)

## <a name="launch-copy-wizard"></a>Inicio del Asistente para copia
1. En la hoja Factoría de datos, haga clic en **Copiar datos** para iniciar el **Asistente para copia**. 
   
   > [!NOTE]
   > Si ve que el explorador web se queda bloqueado en "Autorizando...", deshabilite o desactive la opción **Bloquear cookies y datos de sitios de terceros** en la configuración del explorador, o déjela habilitada y cree una excepción para **login.microsoftonline.com** e intente iniciar de nuevo el asistente.
2. En la página **Propiedades** :
   
   1. Escriba **CopyFromBlobToAzureSql** en **Nombre de tarea**.
   2. Escriba una **descripción** (opcional).
   3. Cambie la **Fecha y hora de inicio** y la **Fecha y hora de finalización** para establecer la fecha de finalización en hoy y la fecha de inicio en cinco días antes.  
   4. Haga clic en **Next**.  
      
      ![Herramienta de copia: página Propiedades](./media/data-factory-copy-data-wizard-tutorial/copy-tool-properties-page.png) 
3. En la página **Almacén de datos de origen**, haga clic en el icono **Azure Blob Storage**. Use esta página para especificar el almacén de datos de origen para la tarea de copia. 
   
    ![Herramienta de copia: página de Almacén de datos de origen](./media/data-factory-copy-data-wizard-tutorial/copy-tool-source-data-store-page.png)
4. En la página **Especificar cuenta de Almacenamiento de blobs de Azure** :
   
   1. Escriba **AzureStorageLinkedService** en **Nombre de servicio vinculado**.
   2. Confirme que la opción **De suscripciones de Azure** está seleccionada para **Método de selección de cuenta**.
   3. Selección la **suscripción** de Azure.  
   4. Seleccione una **cuenta de Azure Storage** en la lista de cuentas de Azure Storage disponibles en la suscripción seleccionada. También puede elegir especificar la configuración de la cuenta de almacenamiento manualmente, para lo que debe seleccionar la opción **Especificar manualmente** en **Método de selección de cuenta** y luego hacer clic en **Siguiente**. 
      
      ![Herramienta de copia: Especificar cuenta de Almacenamiento de blobs de Azure](./media/data-factory-copy-data-wizard-tutorial/copy-tool-specify-azure-blob-storage-account.png)
5. En la página **Elegir el archivo o la carpeta de entrada** :
   
   1. Haga doble clic en **adftutorial** (carpeta).
   2. Seleccione **emp.txt** y haga clic en **Elegir**.
      
      ![Captura de pantalla que muestra la opción Elegir del archivo de entrada.](./media/data-factory-copy-data-wizard-tutorial/copy-tool-choose-input-file-or-folder.png)
6. En la página **Choose the input file or folder** (Elegir el archivo o la carpeta de entrada), haga clic en **Next** (Siguiente). No seleccione **Binary copy**(Copia binaria). 
   
    ![Captura de pantalla que muestra la opción Binary copy (Copia binaria) de la entrada.](./media/data-factory-copy-data-wizard-tutorial/chose-input-file-folder.png) 
7. En la página **Configuración de formato de archivo**, verá los delimitadores y el esquema que el asistente detecta automáticamente al analizar el archivo. También puede especificar los delimitadores manualmente a fin de que el Asistente para copia detenga la detección automática o proceda a la invalidación. Haga clic en **Siguiente** después de revisar los delimitadores y obtener una vista previa de los datos. 
   
    ![Herramienta de copia: Configuración de formato de archivo](./media/data-factory-copy-data-wizard-tutorial/copy-tool-file-format-settings.png)  
8. En la página Almacén de datos de destino, seleccione el icono **Azure SQL Database** y haga clic en **Siguiente**.
   
    ![Herramienta de copia: Elegir el almacén de destino](./media/data-factory-copy-data-wizard-tutorial/choose-destination-store.png)
9. En la página **Especificar la base de datos de Azure SQL** :
   
   1. Escriba **AzureSqlLinkedService** en el campo **Nombre de la conexión**.
   2. Confirme que la opción **De suscripciones de Azure** esté seleccionada para **Método de selección de servidor y base de datos**.
   3. Selección la **suscripción** de Azure.  
   4. Seleccione **Nombre de servidor** y **Base de datos**.
   5. En **Nombre de usuario** y **Contraseña**, escriba los valores pertinentes.
   6. Haga clic en **Next**.  
      
      ![Herramienta de copia: especificación de Azure SQL Database](./media/data-factory-copy-data-wizard-tutorial/specify-azure-sql-database.png)
10. En la página **Asignación de tabla**, seleccione **emp** en la lista desplegable del campo **Destino** y haga clic en **flecha abajo** (opcional) para ver el esquema y obtener una vista previa de los datos.
    
     ![Herramienta de copia: Asignación de tabla](./media/data-factory-copy-data-wizard-tutorial/copy-tool-table-mapping-page.png) 
11. En la página **Asignación de esquema**, haga clic en **Siguiente**.
    
    ![Herramienta de copia: Asignación de esquemas](./media/data-factory-copy-data-wizard-tutorial/schema-mapping-page.png)
12. En la página **Configuración de rendimiento**, haga clic en **Siguiente**. 
    
    ![Captura de pantalla que muestra la página Performance settings (Configuración de rendimiento), donde puede seleccionar Siguiente.](./media/data-factory-copy-data-wizard-tutorial/performance-settings.png)
13. Revise la información de la página **Resumen** y haga clic en **Finalizar**. El asistente crea dos servicios vinculados, dos conjuntos de datos (entrada y salida) y una canalización en la factoría de datos (desde donde se inició al Asistente para copia). 
    
    ![Captura de pantalla que muestra el panel Resumen, donde puede seleccionar Siguiente.](./media/data-factory-copy-data-wizard-tutorial/summary-page.png)

## <a name="launch-monitor-and-manage-application"></a>Inicio de la aplicación de supervisión y administración
1. En la página **Implementación**, haga clic en el vínculo: `Click here to monitor copy pipeline`.
   
   ![Herramienta de copia: Implementación correcta](./media/data-factory-copy-data-wizard-tutorial/copy-tool-deployment-succeeded.png)  
2. La aplicación de supervisión se inicia en una pestaña independiente del explorador web.   
   
   ![Aplicación de supervisión](./media/data-factory-copy-data-wizard-tutorial/monitoring-app.png)   
3. Haga clic en el botón **Actualizar** en la lista **VENTANAS DE ACTIVIDAD** en la parte inferior para ver el estado más reciente en segmentos horarios. Verá cinco ventanas de la actividad de los cinco días entre los tiempos de inicio y finalización para la canalización. La lista no se actualiza automáticamente, por lo que deberá hacer clic en Actualizar un par de veces antes de ver todas las ventanas de actividad en estado Listo. 
4. Seleccione una ventana de actividad en la lista. Vea los detalles en el **Explorador de ventanas de actividad**, situado a la derecha.

    ![Detalles de ventana de actividad](media/data-factory-copy-data-wizard-tutorial/activity-window-details.png)    

    Observe que las fechas 11, 12, 13, 14 y 15 están en color verde, lo que significa que los segmentos de salida diarios para estas fechas ya se han producido. También verá este código de color en la canalización y en el conjunto de datos de salida en la vista de diagrama. En el paso anterior, observe que ya se han producido dos segmentos, uno de los segmentos se está procesando y los otros dos están a la espera de ser procesados (según la codificación de color). 

    Para más información sobre el uso de esta aplicación, consulte el artículo [Supervisión y administración de canalizaciones mediante la aplicación de supervisión](data-factory-monitor-manage-app.md).

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha usado Azure Blob Storage como almacén de datos de origen y Azure SQL Database como almacén de datos de destino en una operación de copia. En la tabla siguiente se proporciona una lista de almacenes de datos que se admiten como orígenes y destinos de la actividad de copia: 

[!INCLUDE [data-factory-supported-data-stores](includes/data-factory-supported-data-stores.md)]

Si quiere más información acerca de las propiedades o campos que se ven en el Asistente para copia de un almacén de datos, haga clic en el vínculo del almacén de datos en la tabla. 