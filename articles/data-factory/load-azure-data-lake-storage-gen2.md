---
title: Carga de datos en Azure Data Lake Storage Gen2
description: Uso de Azure Data Factory para copiar datos en Azure Data Lake Storage Gen2
ms.author: jianleishen
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 02/18/2021
ms.openlocfilehash: dc71dbb50478494b30bb78f4ae55a7f4a6423d62
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109488634"
---
# <a name="load-data-into-azure-data-lake-storage-gen2-with-azure-data-factory"></a>Carga de datos en Azure Data Lake Storage Gen2 con Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

‎Azure Data Lake Storage Gen2 es un conjunto de funcionalidades dedicadas al análisis de macrodatos basado en [Azure Blob Storage](../storage/blobs/storage-blobs-introduction.md). Permite establecer una conexión con los datos usando tanto el sistema de archivos como el almacenamiento de objetos.

Azure Data Factory (ADF) es un servicio de integración de datos en la nube totalmente administrado. Puede utilizar el servicio para rellenar el lago con datos de un amplio conjunto de almacenes de datos locales y basados en la nube y ahorrar tiempo al crear las soluciones de análisis. Para una lista detallada de conectores admitidos, consulte la tabla de [Almacenes de datos admitidos](copy-activity-overview.md#supported-data-stores-and-formats).

Azure Data Factory ofrece una solución de movimiento de datos administrados y de escalabilidad horizontal. Debido a la arquitectura con escalabilidad horizontal de ADF, puede ingerir datos con un alto rendimiento. Para más información, consulte el [rendimiento de la actividad de copia](copy-activity-performance.md).

En este artículo se muestra cómo utilizar la herramienta Copiar datos de Data Factory para cargar datos del _servicio Amazon Web Services S3_ en _Azure Data Lake Storage Gen2_. Puede seguir los mismos pasos para copiar datos de otros tipos de almacenes de datos.

>[!TIP]
>Para copiar datos desde Azure Data Lake Storage Gen1 en Gen2, consulte [en este tutorial específico](load-azure-data-lake-storage-gen2-from-gen1.md).

## <a name="prerequisites"></a>Requisitos previos

* Suscripción de Azure: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Cuenta de Azure Storage con Data Lake Storage Gen2 habilitado: Si no tiene una cuenta de Storage, [debe crear una](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM).
* Cuenta de AWS con un cubo de S3 que contiene datos: En este artículo se muestra cómo copiar datos de Amazon S3. Puede usar otros almacenes de datos siguiendo los mismos pasos.

## <a name="create-a-data-factory"></a>Crear una factoría de datos

1. En el menú de la izquierda, seleccione **Crear un recurso** > **Integración** > **Data Factory**:
   
   ![Selección de la factoría de datos en el panel Nuevo](./media/doc-common-process/new-azure-data-factory-menu.png)

2. En la página **New data factory** (Nueva factoría de datos), proporcione valores para los siguientes campos:
 
    * **Name**: escriba un nombre único global para la factoría de datos de Azure. Si recibe el error "Data factory name *SuNombreDeFactoríaDeDatos* is not available" (El nombre de la factoría de datos SuNombreDeFactoríaDeDatos no está disponible), escriba otro nombre. Por ejemplo, podría utilizar el nombre _**suNombre**_**ADFTutorialDataFactory**. Intente crear de nuevo la factoría de datos. Para conocer las reglas de nomenclatura de los artefactos de Data Factory, consulte [Data Factory: reglas de nomenclatura](naming-rules.md).
    * **Suscripción**: seleccione la suscripción de Azure donde desea crear la factoría de datos. 
    * **Grupo de recursos**: seleccione un grupo de recursos existente en la lista desplegable o seleccione la opción **Crear nuevo** y escriba el nombre de un grupo de recursos. Para obtener más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](../azure-resource-manager/management/overview.md).  
    * **Versión**: seleccione **V2**.
    * **Ubicación**: Seleccione la ubicación de la factoría de datos. Solo las ubicaciones admitidas se muestran en la lista desplegable. Los almacenes de datos que las factorías de datos usan pueden estar en otras ubicaciones y regiones. 

3. Seleccione **Crear**.

4. Una vez completada la creación, vaya a la factoría de datos. Verá la página principal de **Factoría de datos**, tal y como se muestra en la siguiente imagen: 
   
   :::image type="content" source="./media/doc-common-process/data-factory-home-page.png" alt-text="Página principal de Azure Data Factory, con el icono de creación y supervisión.":::

   Seleccione el icono **Author & Monitor** (Creación y supervisión) para iniciar la aplicación de integración de datos en una pestaña independiente.

## <a name="load-data-into-azure-data-lake-storage-gen2"></a>Carga de datos en Azure Data Lake Storage Gen2

1. En la página **Introducción**, seleccione el icono **Copiar datos** para iniciar la herramienta para copiar datos.

2. En la página **Properties** (Propiedades), especifique **CopyFromAmazonS3ToADLS** en el campo **Task name** (Nombre de la tarea) y seleccione **Next** (Siguiente).

    ![Página de propiedades](./media/load-azure-data-lake-storage-gen2/copy-data-tool-properties-page.png)
3. En la página **Almacén de datos de origen**, haga clic en **+ Crear nueva conexión**. Seleccione **Amazon S3** en la galería de conectores y seleccione **Continue** (Continuar).
    
    ![Página del almacén de datos de origen S3](./media/load-azure-data-lake-storage-gen2/source-data-store-page-s3.png)
    
4. En la página **New linked service (Amazon S3)** [Nuevo servicio vinculado (Amazon S3)], realice los pasos siguientes:

   1. Especifique el valor de **Access Key ID** (Identificador de clave de acceso).
   2. Especifique el valor de **Secret Access Key** (Clave de acceso secreta).
   3. Haga clic en **Prueba de conexión** para validar la configuración y, después, seleccione **Crear**.

      ![Especificación de la cuenta de Amazon S3](./media/load-azure-data-lake-storage-gen2/specify-amazon-s3-account.png)
   4. Podrá ver una nueva conexión de Amazon S3 creada. Seleccione **Next** (Siguiente). 

5. En la página **Choose the input file or folder** (Elegir archivo o carpeta de entrada), vaya a la carpeta y el archivo que desea copiar. Seleccione la carpeta o el archivo, y, después, seleccione **Choose** (Elegir).

    ![Elegir archivo o carpeta de entrada](./media/load-azure-data-lake-storage-gen2/choose-input-folder.png)

6. Elija el comportamiento de copia; para ello, seleccione las opciones **Recursively** (Recursivamente) y **Binary copy** (Copia binaria). Seleccione **Next** (Siguiente).

    ![Captura de pantalla que muestra la opción para elegir el archivo o la carpeta de entrada donde puede optar por copiar el archivo de forma recursiva o realizar una copia binaria.](./media/load-azure-data-lake-storage-gen2/specify-binary-copy.png)
    
7. En la página **Destination data store** (Almacén de datos de destino), haga clic en **+ Create new connection** (+ Crear nueva conexión) y después seleccione **Azure Data Lake Storage Gen2** y **Continue** (Continuar).

    ![Página Destination data store (Almacén de datos de destino)](./media/load-azure-data-lake-storage-gen2/destination-data-storage-page.png)

8. En la página **New Linked Service (Azure Data Lake Storage Gen2)** [Nuevo servicio vinculado (Azure Data Lake Storage Gen2)], siga estos pasos:

   1. Seleccione la cuenta habilitada para Data Lake Storage Gen2 en la lista desplegable "Storage account name" (Nombre de la cuenta de almacenamiento).
   2. Seleccione **Create** (Crear) para crear la conexión. Luego, seleccione **Siguiente**.   

        ![Especificación de una cuenta de Azure Data Lake Storage Gen2](./media/load-azure-data-lake-storage-gen2/specify-azure-data-lake-storage.png)

9. En la página **Choose the output file or folder** (Elegir archivo o carpeta de salida) escriba **copyfroms3** como nombre de la carpeta de salida y seleccione **Siguiente**. ADF creará el sistema de archivos y subcarpetas correspondientes de ADLS Gen2 durante la copia, si no existen.

    ![Captura de pantalla que muestra la ruta de acceso de la carpeta que ha especificado.](./media/load-azure-data-lake-storage-gen2/specify-adls-path.png)

10. En la página **Settings** (Configuración), seleccione **Next** (Siguiente) para usar las configuraciones predeterminadas.

    ![Página Configuración](./media/load-azure-data-lake-storage-gen2/copy-settings.png)

11. En la página **Resumen**, revise la configuración y seleccione **Siguiente**.

    ![Página de resumen](./media/load-azure-data-lake-storage-gen2/copy-summary.png)

12. En la página **Deployment** (Implementación), seleccione **Monitor** (Supervisión) para supervisar la canalización (tarea). 
 
13. Cuando la ejecución de la canalización se complete correctamente, verá una ejecución de canalización que se desencadena mediante un desencadenador manual. Puede usar los vínculos de la columna **PIPELINE NAME** (Nombre de la canalización) para ver los detalles de la actividad y volver a ejecutar la canalización.

    ![La supervisión de la canalización se ejecuta](./media/load-azure-data-lake-storage-gen2/monitor-pipeline-runs.png)

14. Para ver las ejecuciones de actividad asociadas a la ejecución de la canalización, seleccione el vínculo **CopyFromAmazonS3ToADLS** en la columna PIPELINE NAME (Nombre de la canalización). Para más información sobre la operación de copia, seleccione el vínculo **Details** (Detalles) (icono de gafas) en la columna ACTIVITY NAME (Nombre de actividad). Puede supervisar detalles como el volumen de datos copiados desde el origen al receptor, el rendimiento de los datos, los pasos de ejecución con su duración correspondiente y las configuraciones que se utilizan.
 
    ![Supervisión de las ejecuciones de actividad](./media/load-azure-data-lake-storage-gen2/monitor-activity-runs.png)
    
    ![Detalles de la supervisión de la ejecución de actividad](./media/load-azure-data-lake-storage-gen2/monitor-activity-run-details.png)

15. Para actualizar la vista, seleccione Refresh (Actualizar). Para volver a la vista Ejecuciones de canalización, seleccione **All pipeline runs** (Todas las ejecuciones de canalización) en la parte superior.

16. Verifique que los datos se copian en la cuenta de Data Lake Storage Gen2.

## <a name="next-steps"></a>Pasos siguientes

* [Información general de la actividad de copia](copy-activity-overview.md)
* [Conector de Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md)
