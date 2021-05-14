---
title: Copia de datos desde Azure Data Lake Storage Gen1 a Gen2
description: Uso de Azure Data Factory para copiar datos de Azure Data Lake Storage Gen1 en Gen2
ms.author: jianleishen
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 02/18/2021
ms.openlocfilehash: 1af7e5ea1038d78250271a8f9a2c69316e66883a
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109488652"
---
# <a name="copy-data-from-azure-data-lake-storage-gen1-to-gen2-with-azure-data-factory"></a>Copia de datos de Azure Data Lake Storage Gen1 en Gen2 con Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Azure Data Lake Storage Gen2 es un conjunto de funciones dedicadas al análisis de macrodatos compilado en [Azure Blob Storage](../storage/blobs/storage-blobs-introduction.md). Puede usarlo para interactuar con los datos usando el paradigma de sistema de archivos o el de almacenamiento de objetos.

Si actualmente usa Azure Data Lake Storage Gen1, puede evaluar Azure Data Lake Storage Gen2 mediante la copia de datos de Data Lake Storage Gen1 en Gen2 con Azure Data Factory.

Azure Data Factory es un servicio de integración de datos en la nube totalmente administrado. Puede utilizar el servicio para rellenar el lago con datos de un amplio conjunto de almacenes de datos locales y basados en la nube y, de este modo, ahorrar tiempo cuando compile las soluciones de análisis. Para obtener una lista de conectores admitidos, vea la tabla de [Almacenes de datos admitidos](copy-activity-overview.md#supported-data-stores-and-formats).

Azure Data Factory ofrece una solución de movimiento de datos administrados y de escalabilidad horizontal. Debido a la arquitectura con escalabilidad horizontal de Data Factory, puede ingerir datos con un alto rendimiento. Para obtener más información, vea el artículo [Copiar rendimiento de actividad](copy-activity-performance.md).

En este artículo se muestra cómo utilizar la herramienta Copiar datos de Data Factory para copiar datos de Azure Data Lake Storage Gen1 en Azure Data Lake Storage Gen2. Puede seguir los mismos pasos para copiar datos de otros tipos de almacenes de datos.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Cuenta de Azure Data Lake Storage Gen1 con datos en ella.
* Cuenta de Azure Storage con Data Lake Storage Gen2 habilitado. Si no tiene una cuenta de Storage, [debe crear una](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM).

## <a name="create-a-data-factory"></a>Crear una factoría de datos

1. En el menú de la izquierda, seleccione **Crear un recurso** > **Datos y análisis** > **Data Factory**.
   
   ![Selección de la factoría de datos en el panel Nuevo](./media/quickstart-create-data-factory-portal/new-azure-data-factory-menu.png)

2. En la página **Nueva factoría de datos**, proporcione los valores de los campos que se muestran en la imagen siguiente: 
      
   ![Página nueva de la factoría de datos](./media/load-azure-data-lake-storage-gen2-from-gen1/new-azure-data-factory.png)
 
    * **Name**: escriba un nombre único global para la factoría de datos de Azure. Si recibe el error "El nombre de la factoría de datos \"LoadADLSDemo\" no está disponible", escriba uno diferente. Por ejemplo, utilice _**suNombre**_ **ADFTutorialDataFactory**. Cree la factoría de datos otra vez. Para conocer las reglas de nomenclatura de los artefactos de Data Factory, consulte [Data Factory: reglas de nomenclatura](naming-rules.md).
    * **Suscripción**: seleccione la suscripción de Azure donde desea crear la factoría de datos. 
    * **Grupo de recursos**: Seleccione un grupo de recursos existente de la lista desplegable. También puede seleccionar la opción **Crear** y escribir el nombre de un grupo de recursos. Para más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](../azure-resource-manager/management/overview.md). 
    * **Versión**: seleccione **V2**.
    * **Ubicación**: Seleccione la ubicación de la factoría de datos. Solo las ubicaciones admitidas se muestran en la lista desplegable. Los almacenes de datos que las factorías de datos usan pueden estar en otras ubicaciones y regiones. 

3. Seleccione **Crear**.
4. Una vez completada la creación, vaya a la factoría de datos. Verá la página principal de **Factoría de datos**, tal y como se muestra en la siguiente imagen: 
   
   ![Página principal Factoría de datos](./media/load-azure-data-lake-storage-gen2-from-gen1/data-factory-home-page.png)

5. Seleccione el icono **Creación y supervisión** para iniciar la aplicación de integración de datos en una pestaña independiente.

## <a name="load-data-into-azure-data-lake-storage-gen2"></a>Carga de datos en Azure Data Lake Storage Gen2

1. En la página **Introducción**, seleccione el icono **Copiar datos** para iniciar la herramienta para copiar datos. 

   ![Icono de la herramienta Copiar datos](./media/load-azure-data-lake-storage-gen2-from-gen1/copy-data-tool-tile.png)
2. En la página **Propiedades**, especifique **CopyFromADLSGen1ToGen2** en el campo **Nombre de la tarea**. Seleccione **Next** (Siguiente).

    ![Página de propiedades](./media/load-azure-data-lake-storage-gen2-from-gen1/copy-data-tool-properties-page.png)
3. En la página **Almacén de datos de origen**, haga clic en **+ Crear una conexión nueva**.

    ![Página Source data store (Almacén de datos de origen)](./media/load-azure-data-lake-storage-gen2-from-gen1/source-data-store-page.png)
    
4. Seleccione **Azure Data Lake Storage Gen1** en la galería de conectores y seleccione **Continuar**.
    
    ![Página de Azure Data Lake Storage Gen1 de almacén de datos de origen](./media/load-azure-data-lake-storage-gen2-from-gen1/source-data-store-page-adls-gen1.png)
    
5. En la página **Especificar conexión de Azure Data Lake Storage Gen1**, siga estos pasos:

   a. Seleccione su instancia de Data Lake Storage Gen1 para el nombre de cuenta y especifique o valide el **inquilino**.
  
   b. Seleccione **Prueba de conexión** para validar la configuración. Después, seleccione **Finalizar**.
  
   c. Verá que se ha creado una nueva conexión. Seleccione **Next** (Siguiente).
   
   > [!IMPORTANT]
   > En este tutorial se utilizará una identidad administrada para recursos de Azure con el fin de autenticar la cuenta de Azure Data Lake Storage Gen1. Para conceder a la identidad administrada los permisos adecuados en Azure Data Lake Storage Gen1, siga [estas instrucciones](connector-azure-data-lake-store.md#managed-identity).
   
   ![Especificación de una cuenta de Azure Data Lake Storage Gen1](./media/load-azure-data-lake-storage-gen2-from-gen1/specify-adls-gen1-account.png)
      
6. En la página **Elegir archivo o carpeta de entrada**, vaya a la carpeta y el archivo que quiere copiar. Seleccione la carpeta o el archivo y seleccione **Elegir**.

    ![Elegir archivo o carpeta de entrada](./media/load-azure-data-lake-storage-gen2-from-gen1/choose-input-folder.png)

7. Especifique el comportamiento de copia seleccionando las opciones **Copiar archivos de forma recursiva** y **Copia binaria**. Seleccione **Next** (Siguiente).

    ![Captura de pantalla que muestra la opción para elegir el archivo o la carpeta de entrada donde puede optar por copiar el archivo de forma recursiva o realizar una copia binaria.](./media/load-azure-data-lake-storage-gen2-from-gen1/specify-binary-copy.png)
    
8. En la página **Almacén de datos de destino**, seleccione **+ Crear una conexión nueva** > **Azure Data Lake Storage Gen2** > **Continuar**.

    ![Página Destination data store (Almacén de datos de destino)](./media/load-azure-data-lake-storage-gen2-from-gen1/destination-data-storage-page.png)

9. En la página **Especificar conexión de Azure Data Lake Storage Gen2**, siga estos pasos:

   a. Seleccione la cuenta habilitada para Data Lake Storage Gen2 en la lista desplegable **Nombre de la cuenta de almacenamiento**.
   
   b. Seleccione **Finalizar** para crear la conexión. Luego, seleccione **Siguiente**.
   
   ![Especificación de una cuenta de Azure Data Lake Storage Gen2](./media/load-azure-data-lake-storage-gen2-from-gen1/specify-adls-gen2-account.png)

10. En la página **Elegir archivo o carpeta de salida** escriba **copyfromadlsgen1** como nombre de la carpeta de salida y seleccione **Siguiente**. Si no existen el sistema de archivos de Azure Data Lake Storage Gen2 correspondiente y las subcarpetas, Data Factory los crea durante la copia.

    ![Captura de pantalla que muestra la ruta de acceso de la carpeta que ha especificado.](./media/load-azure-data-lake-storage-gen2-from-gen1/specify-adls-gen2-path.png)

11. En la página **Configuración**, seleccione **Siguiente** para usar la configuración predeterminada.

12. En la página **Resumen**, revise la configuración y seleccione **Siguiente**.

    ![Página de resumen](./media/load-azure-data-lake-storage-gen2-from-gen1/copy-summary.png)
13. En la página **Implementación**, seleccione **Supervisión** para supervisar la canalización.

    ![Página Deployment (Implementación)](./media/load-azure-data-lake-storage-gen2-from-gen1/deployment-page.png)
14. Observe que la pestaña **Monitor** (Supervisión) de la izquierda se selecciona automáticamente. La columna **Actions** (Acciones) incluye los vínculos para ver los detalles de la ejecución de actividad y volver a ejecutar la canalización.

    ![La supervisión de la canalización se ejecuta](./media/load-azure-data-lake-storage-gen2-from-gen1/monitor-pipeline-runs.png)

15. Para ver las ejecuciones de actividad asociadas con la ejecución de canalización, seleccione el vínculo **View Activity Runs** (Ver ejecuciones de actividad) en la columna **Actions** (Acciones). Como solo hay una actividad (actividad de copia) en la canalización, solo verá una entrada. Para volver a la vista de ejecuciones de canalización, seleccione el vínculo **Pipelines** (Canalizaciones) de la parte superior. Seleccione **Refresh** (Actualizar) para actualizar la lista. 

    ![Supervisión de las ejecuciones de actividad](./media/load-azure-data-lake-storage-gen2-from-gen1/monitor-activity-runs.png)

16. Para supervisar los detalles de la ejecución de cada actividad de copia, seleccione el vínculo con un icono de gafas **Details** (Detalles) en **Actions** (Acciones) en la vista de supervisión de la actividad. Puede supervisar detalles como el volumen de datos copiados desde el origen al receptor, el rendimiento de los datos, los pasos de ejecución con su duración correspondiente y las configuraciones que se utilizan.

    ![Detalles de la supervisión de la ejecución de actividad](./media/load-azure-data-lake-storage-gen2-from-gen1/monitor-activity-run-details.png)

17. Verifique que los datos se copian en la cuenta de Azure Data Lake Storage Gen2.

## <a name="best-practices"></a>Procedimientos recomendados

Para evaluar la actualización de Azure Data Lake Storage Gen1 a Azure Data Lake Storage Gen2 en general, vea [Actualizar sus soluciones de análisis de macrodatos de Azure Data Lake Storage Gen1 a Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-migrate-gen1-to-gen2.md). Las secciones siguientes presentan procedimientos recomendados para usar Data Factory para una actualización de datos de Data Lake Storage Gen1 a Data Lake Storage Gen2.

### <a name="data-partition-for-historical-data-copy"></a>Partición de datos para una copia de datos históricos

- Si el tamaño total de los datos en Data Lake Storage Gen1 es inferior a 30 TB y el número de archivos es menos de 1 millón, puede copiar todos los datos en una ejecución de actividad de copia única.
- Si tiene una mayor cantidad de datos para copiar, o si quiere flexibilidad para administrar la migración de datos por lotes y hacer que cada uno de estos se complete en un periodo de tiempo determinado, particione los datos. La creación de particiones también reduce el riesgo de se produzca una incidencia inesperada.

Utilice una prueba de concepto para comprobar la solución de un extremo a otro y probar el rendimiento de la copia en su entorno. Pasos principales de la prueba de concepto: 

1. Crear una canalización de Data Factory con una actividad de copia única para copiar varios TB de datos de Data Lake Storage Gen1 a Data Lake Storage Gen2 con el fin de obtener una base de referencia de rendimiento de copia. Empiece con las [unidades de integración de datos (DIU)](copy-activity-performance-features.md#data-integration-units) en 128. 
2. En función del rendimiento de copia que se obtenga en el paso 1, calcular el tiempo estimado necesario para toda la migración de datos. 
3. (Opcional) Crear una tabla de control y definir el filtro de archivos para crear particiones en los archivos que se van a migrar. La manera de crear particiones en los archivos es la siguiente: 

    - Partición mediante el nombre de carpeta o el nombre de carpeta con un filtro de comodín. Se recomienda este método.
    - Partición mediante la hora de última modificación de un archivo.

### <a name="network-bandwidth-and-storage-io"></a>E/S de almacenamiento y ancho de banda de red 

Puede controlar la simultaneidad de los trabajos de copia de Data Factory que leen datos de Data Lake Storage Gen1 y escriben datos en Data Lake Storage Gen2. De esta manera, puede administrar el uso en esa E/S de almacenamiento para evitar que afecte al trabajo normal del negocio en Data Lake Storage Gen1 durante la migración.

### <a name="permissions"></a>Permisos 

En Data Factory, el [conector de Data Lake Storage Gen1](connector-azure-data-lake-store.md) admite la entidad de servicio y la identidad administrada para las autenticaciones de recursos de Azure. El [conector de Data Lake Storage Gen2](connector-azure-data-lake-storage.md) admite la clave de cuenta, la entidad de servicio y la identidad administrada para las autenticaciones de recursos de Azure. Para hacer que Data Factory pueda navegar y copiar todos los archivos o listas de control de acceso (ACL) que necesita, conceda los permisos suficientemente elevados para la cuenta a la que se proporciona acceso, lectura o escritura de todos los archivos y establezca las ACL si quiere. Conceda un rol de superusuario o propietario durante la migración. 

### <a name="preserve-acls-from-data-lake-storage-gen1"></a>Conservación de ACL desde Data Lake Storage Gen1

Si quiere replicar las ACL junto con los archivos de datos cuando se actualiza de Data Lake Storage Gen1 a Data Lake Storage Gen2, vea [Conservar las ACL de Data Lake Storage Gen1](connector-azure-data-lake-storage.md#preserve-acls). 

### <a name="incremental-copy"></a>Copia incremental 

Puede usar varios métodos para cargar únicamente los archivos nuevos o actualizados desde Data Lake Storage Gen1:

- Cargar archivos nuevos o actualizados mediante una carpeta o nombres de archivo con particiones por hora. Un ejemplo es /2019/05/13/*.
- Cargar archivos nuevos o actualizados por LastModifiedDate.
- Identificar archivos nuevos o actualizados mediante cualquier herramienta o solución de terceros. Después, pasar el nombre de archivo o carpeta a la canalización de Data Factory a través de un parámetro, o una tabla o archivo. 

La frecuencia adecuada para realizar una carga incremental depende del número total de archivos en Azure Data Lake Storage Gen1 y del volumen de archivos nuevos o actualizados que se van a cargar cada vez. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Introducción de la actividad de copia](copy-activity-overview.md)
> [Conector de Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md)
> [Conector de Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md)