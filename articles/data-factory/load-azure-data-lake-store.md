---
title: Carga de datos en Azure Data Lake Storage Gen1
description: Uso de Azure Data Factory para copiar datos en Azure Data Lake Storage Gen1
ms.author: jianleishen
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 02/18/2021
ms.openlocfilehash: 5269267f0fa786163b6c1fa1a4d4bfc8d0ff9af5
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109483522"
---
# <a name="load-data-into-azure-data-lake-storage-gen1-by-using-azure-data-factory"></a>Carga de datos en Azure Data Lake Storage Gen1 mediante Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

[Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-overview.md) (anteriormente conocido como Azure Data Lake Store) es un repositorio de gran escala en toda la empresa para cargas de trabajo de análisis de macrodatos. Azure Data Lake Storage Gen1 permite capturar datos de cualquier tamaño, tipo y velocidad de ingesta. Los datos se capturan en un único lugar para análisis operativos y exploratorios.

Azure Data Factory es un servicio de integración de datos en la nube totalmente administrado. Puede utilizar el servicio para rellenar la instancia de Data Lake Store con datos del sistema actual y ahorrar tiempo al compilar las soluciones de análisis.

Azure Data Factory ofrece las siguientes ventajas para cargar datos en Data Lake Storage Gen1:

* **Fácil de configurar**: con un asistente intuitivo en cinco pasos sin necesidad de scripting.
* **Amplia compatibilidad para el almacenamiento de datos**: compatibilidad integrada para un amplio conjunto de almacenes de datos tanto locales como en la nube. Para una lista detallada, consulte la tabla de [almacenes de datos admitidos](copy-activity-overview.md#supported-data-stores-and-formats).
* **Seguro y compatible**: los datos se transfieren a través de HTTPS o ExpressRoute. La presencia del servicio global garantiza que los datos nunca abandonan el límite geográfico.
* **Alto rendimiento**: hasta 1 GB/s de velocidad de carga de datos en Data Lake Storage Gen1. Para más información, consulte el [rendimiento de la actividad de copia](copy-activity-performance.md).

En este artículo se muestra cómo utilizar la herramienta Copy Data de Data Factory para _cargar datos de Amazon S3 en Data Lake Storage Gen1_. Puede seguir los mismos pasos para copiar datos de otros tipos de almacenes de datos.

> [!NOTE]
> Para más información, vea [Copia de datos con Data Lake Storage Gen1 como origen o destino mediante Azure Data Factory](connector-azure-data-lake-store.md).

## <a name="prerequisites"></a>Requisitos previos

* Suscripción de Azure: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Cuenta de Data Lake Storage Gen1: si no tiene una cuenta de Data Lake Storage Gen1, consulte las instrucciones de [Creación de una cuenta de Data Lake Storage Gen1](../data-lake-store/data-lake-store-get-started-portal.md#create-a-data-lake-storage-gen1-account).
* Amazon S3: En este artículo se muestra cómo copiar datos de Amazon S3. Puede usar otros almacenes de datos siguiendo los mismos pasos.

## <a name="create-a-data-factory"></a>Crear una factoría de datos

1. En el menú de la izquierda, seleccione **Crear un recurso** > **Analytics** > **Data Factory**:
   
   ![Selección de la factoría de datos en el panel Nuevo](./media/quickstart-create-data-factory-portal/new-azure-data-factory-menu.png)

2. En la página **Nueva factoría de datos**, proporcione los valores de los campos que se muestran en la imagen siguiente: 
      
   ![Página New data factory (Nueva factoría de datos)](./media/load-data-into-azure-data-lake-store//new-azure-data-factory.png)
 
    * **Name**: escriba un nombre único global para la factoría de datos de Azure. Si recibe el error "El nombre de la factoría de datos \"LoadADLSG1Demo\" no está disponible", escriba uno diferente. Por ejemplo, podría utilizar el nombre _**suNombre**_**ADFTutorialDataFactory**. Intente crear de nuevo la factoría de datos. Para conocer las reglas de nomenclatura de los artefactos de Data Factory, consulte [Data Factory: reglas de nomenclatura](naming-rules.md).
    * **Suscripción**: seleccione la suscripción de Azure donde desea crear la factoría de datos. 
    * **Grupo de recursos**: seleccione un grupo de recursos existente en la lista desplegable o seleccione la opción **Crear nuevo** y escriba el nombre de un grupo de recursos. Para obtener más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](../azure-resource-manager/management/overview.md).  
    * **Versión**: seleccione **V2**.
    * **Ubicación**: Seleccione la ubicación de la factoría de datos. Solo las ubicaciones admitidas se muestran en la lista desplegable. Los almacenes de datos que las factorías de datos usan pueden estar en otras ubicaciones y regiones. Estos almacenes de datos incluyen Azure Data Lake Storage Gen1, Azure Storage, Azure SQL Database, etc.

3. Seleccione **Crear**.
4. Una vez completada la creación, vaya a la factoría de datos. Verá la página principal de **Factoría de datos**, tal y como se muestra en la siguiente imagen: 
   
   ![Página principal Factoría de datos](./media/load-data-into-azure-data-lake-store/data-factory-home-page.png)

   Seleccione el icono **Author & Monitor** (Creación y supervisión) para iniciar la aplicación de integración de datos en una pestaña independiente.

## <a name="load-data-into-data-lake-storage-gen1"></a>Carga de datos en Data Lake Storage Gen1

1. En la página **Get started** (Introducción), seleccione el icono **Copy Data** (Copiar datos) para iniciar la herramienta para copiar datos: 

   ![Icono de la herramienta Copy Data](./media/load-data-into-azure-data-lake-store/copy-data-tool-tile.png)
2. En la página **Properties** (Propiedades), especifique **CopyFromAmazonS3ToADLS** en el campo **Task name** (Nombre de la tarea) y seleccione **Next** (Siguiente):

    ![Página de propiedades](./media/load-data-into-azure-data-lake-store/copy-data-tool-properties-page.png)
3. En la página **Almacén de datos de origen**, haga clic en **+ Crear nueva conexión**:

    ![Página Source data store (Almacén de datos de origen)](./media/load-data-into-azure-data-lake-store/source-data-store-page.png)
    
    Seleccione **Amazon S3** y, luego, seleccione **Continuar**.
    
    ![Página del almacén de datos de origen S3](./media/load-data-into-azure-data-lake-store/source-data-store-page-s3.png)
    
4. En la página **Specify Amazon S3 connection** (Especificar conexión de Amazon S3), siga estos pasos: 
   1. Especifique el valor de **Access Key ID** (Identificador de clave de acceso).
   2. Especifique el valor de **Secret Access Key** (Clave de acceso secreta).
   3. Seleccione **Finalizar**.
   
      ![Captura de pantalla que muestra el panel Nuevo servicio vinculado donde puede especificar los valores.](./media/load-data-into-azure-data-lake-store/specify-amazon-s3-account.png)
   
   4. Podrá ver una nueva conexión. Seleccione **Next** (Siguiente).
   
   ![Captura de pantalla que muestra la nueva conexión.](./media/load-data-into-azure-data-lake-store/specify-amazon-s3-account-created.png)
   
5. En la página **Choose the input file or folder** (Elegir archivo o carpeta de entrada), vaya a la carpeta y el archivo que desea copiar. Seleccione la carpeta o el archivo, **Choose** (Elegir) y **Next** (Siguiente):

    ![Elegir archivo o carpeta de entrada](./media/load-data-into-azure-data-lake-store/choose-input-folder.png)

6. Elija el comportamiento de copia seleccionando las opciones **Copy files recursively** (Copiar archivos de forma recursiva) y **Binary copy** (Copia binaria). Seleccione **Siguiente**:

    ![Captura de pantalla que muestra la opción para elegir el archivo o la carpeta de entrada donde puede optar por copiar el archivo de forma recursiva o realizar una copia binaria.](./media/load-data-into-azure-data-lake-store/specify-binary-copy.png)
    
7. En la página **Almacén de datos de destino**, haga clic en **+Crear nueva conexión** y después seleccione **Azure Data Lake Storage Gen1** y **Continuar**:

    ![Página Destination data store (Almacén de datos de destino)](./media/load-data-into-azure-data-lake-store/destination-data-storage-page.png)

8. En la página **New Linked Service (Azure Data Lake Storage Gen1)** [Nuevo servicio vinculado (Azure Data Lake Storage Gen1)], siga estos pasos: 

   1. Seleccione la cuenta de Data Lake Storage Gen1 para el **nombre de la cuenta de Data Lake Store**.
   2. Especifique el **inquilino** y seleccione Finalizar.
   3. Seleccione **Next** (Siguiente).
   
   > [!IMPORTANT]
   > En este tutorial, utilizará una identidad administrada para recursos de Azure para autenticar la cuenta de Data Lake Storage Gen1. Asegúrese de conceder a MSI los permisos adecuados en Data Lake Storage Gen1 con [estas instrucciones](connector-azure-data-lake-store.md#managed-identity).
   
   ![Definición de la cuenta de Data Lake Storage Gen1](./media/load-data-into-azure-data-lake-store/specify-adls.png)
9. En la página **Choose the output file or folder** (Elegir archivo o carpeta de salida) escriba **copyfroms3** como nombre de la carpeta de salida y seleccione **Next** (Siguiente): 

    ![Captura de pantalla que muestra la ruta de acceso de la carpeta que ha especificado.](./media/load-data-into-azure-data-lake-store/specify-adls-path.png)

10. En la página **Settings** (Configuración), seleccione **Next** (Siguiente):

    ![Página Configuración](./media/load-data-into-azure-data-lake-store/copy-settings.png)
11. En la página **Summary** (Resumen), revise la configuración y seleccione **Next** (Siguiente):

    ![Página de resumen](./media/load-data-into-azure-data-lake-store/copy-summary.png)
12. En la página **Deployment** (Implementación), seleccione **Monitor** (Supervisión) para supervisar la canalización (tarea):

    ![Página Deployment (Implementación)](./media/load-data-into-azure-data-lake-store/deployment-page.png)
13. Observe que la pestaña **Monitor** (Supervisión) de la izquierda se selecciona automáticamente. La columna **Actions** (Acciones) incluye los vínculos para ver los detalles de la ejecución de actividad y volver a ejecutar la canalización:

    ![La supervisión de la canalización se ejecuta](./media/load-data-into-azure-data-lake-store/monitor-pipeline-runs.png)
14. Para ver las ejecuciones de actividad asociadas con la ejecución de canalización, seleccione el vínculo **View Activity Runs** (Ver ejecuciones de actividad) en la columna **Actions** (Acciones). Como solo hay una actividad (actividad de copia) en la canalización, solo verá una entrada. Para volver a la vista de ejecuciones de canalización, seleccione el vínculo **Pipelines** (Canalizaciones) de la parte superior. Seleccione **Refresh** (Actualizar) para actualizar la lista. 

    ![Supervisión de las ejecuciones de actividad](./media/load-data-into-azure-data-lake-store/monitor-activity-runs.png)

15. Para supervisar los detalles de la ejecución de cada actividad de copia, seleccione el vínculo **Details** (Detalles) en **Actions** (Acciones) en la vista de supervisión de la actividad. Puede supervisar detalles como el volumen de datos copiados desde el origen al receptor, el rendimiento de los datos, los pasos de ejecución con su duración correspondiente y las configuraciones que se utilizan:

    ![Detalles de la supervisión de la ejecución de actividad](./media/load-data-into-azure-data-lake-store/monitor-activity-run-details.png)

16. Verifique que los datos se copian en la cuenta de Data Lake Storage Gen1: 

    ![Verificación de la salida de Data Lake Storage Gen1](./media/load-data-into-azure-data-lake-store/adls-copy-result.png)

## <a name="next-steps"></a>Pasos siguientes

Prosiga en el siguiente artículo para obtener información sobre la compatibilidad de Data Lake Storage Gen1: 

> [!div class="nextstepaction"]
>[Conector de Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md)
