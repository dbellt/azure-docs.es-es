---
title: Carga de datos en Azure Synapse Analytics
description: Uso de Azure Data Factory para copiar datos en Azure Synapse Analytics
ms.author: jianleishen
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 01/29/2020
ms.openlocfilehash: 0a83a80ab8be8081e058436115eed82e6f5e7ce6
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109485574"
---
# <a name="load-data-into-azure-synapse-analytics-by-using-azure-data-factory"></a>Carga de datos en Azure Synapse Analytics mediante Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

[Azure Synapse Analytics](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) es una base de datos de escalabilidad horizontal y basada en la nube capaz de procesar volúmenes masivos de datos (tanto relacionales como no relacionales). Basada en la arquitectura de procesamiento paralelo masivo (MPP), está mejorada para controlar las cargas de trabajo empresariales. Ofrece elasticidad en la nube con la flexibilidad para escalar almacenamiento y proceso de forma independiente.

La introducción a Azure Synapse Analytics es ahora más fácil que nunca si se usa Azure Data Factory. Azure Data Factory es un servicio de integración de datos en la nube totalmente administrado. Puede usar el servicio para rellenar una instancia de Azure Synapse Analytics con los datos del sistema actual y ahorrar tiempo al crear las soluciones de análisis.

Azure Data Factory ofrece los siguientes beneficios para cargar datos en Azure Synapse Analytics:

* **Fácil de configurar**: con un asistente intuitivo en cinco pasos sin necesidad de scripting.
* **Amplia compatibilidad para el almacenamiento de datos**: compatibilidad integrada para un amplio conjunto de almacenes de datos tanto locales como en la nube. Para una lista detallada, consulte la tabla de [almacenes de datos admitidos](copy-activity-overview.md#supported-data-stores-and-formats).
* **Seguro y compatible**: los datos se transfieren a través de HTTPS o ExpressRoute. La presencia del servicio global garantiza que los datos nunca abandonan el límite geográfico.
* **Rendimiento sin precedentes mediante PolyBase**: Polybase es la forma más eficaz de mover datos a Azure Synapse Analytics. Mediante la característica de blob de almacenamiento provisional, puede alcanzar velocidades de carga altas para todos los tipos de almacenes de datos, incluido Azure Blob Storage y Data Lake Store. (Polybase es compatible con Azure Blob Storage y Azure Data Lake Store de forma predeterminada). Para más información, consulte el [rendimiento de la actividad de copia](copy-activity-performance.md).

En este artículo se muestra cómo usar la herramienta Copiar datos de Data Factory a fin de _cargar datos de Azure SQL Database en Azure Synapse Analytics_. Puede seguir los mismos pasos para copiar datos de otros tipos de almacenes de datos.

> [!NOTE]
> Para más información, consulte [Copia de datos con Azure Synapse Analytics como origen o destino mediante Azure Data Factory](connector-azure-sql-data-warehouse.md).

## <a name="prerequisites"></a>Requisitos previos

* Suscripción de Azure: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Azure Synapse Analytics: Este almacén de datos contiene los datos que se copian de SQL Database. Si no tiene una instancia de Azure Synapse Analytics, consulte las instrucciones de [Creación de una instancia de Azure Synapse Analytics](../synapse-analytics/sql-data-warehouse/load-data-from-azure-blob-storage-using-copy.md).
* Azure SQL Database: este tutorial copia los datos del conjunto de datos de ejemplo Adventure Works LT en Azure SQL Database. Puede crear esta base de datos de ejemplo en SQL Database si sigue las instrucciones que aparecen en [Creación de una base de datos de ejemplo en Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md).
* Cuenta de almacenamiento de Azure: Azure Storage se usa como blob de _almacenamiento provisional_ en la operación de copia masiva. Si no dispone de una cuenta de almacenamiento de Azure, consulte las instrucciones de [Creación de una cuenta de almacenamiento](../storage/common/storage-account-create.md).

## <a name="create-a-data-factory"></a>Crear una factoría de datos

1. En el menú de la izquierda, seleccione **Crear un recurso** > **Datos y análisis** > **Data Factory**:

2. En la página **Nueva factoría de datos**, proporcione valores para los siguientes elementos:

    * **Name**: Escriba *LoadSQLDWDemo* como nombre. El nombre de la factoría de datos debe ser *globalmente único. Si recibe el error "El nombre de la factoría de datos 'LoadSQLDWDemo' no está disponible", escriba otro nombre. Por ejemplo, podría utilizar el nombre _**suNombre**_**ADFTutorialDataFactory**. Intente crear de nuevo la factoría de datos. Para conocer las reglas de nomenclatura de los artefactos de Data Factory, consulte [Data Factory: reglas de nomenclatura](naming-rules.md).
    * **Suscripción**: seleccione la suscripción de Azure donde desea crear la factoría de datos. 
    * **Grupo de recursos**: seleccione un grupo de recursos existente en la lista desplegable o seleccione la opción **Crear nuevo** y escriba el nombre de un grupo de recursos. Para obtener más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](../azure-resource-manager/management/overview.md).  
    * **Versión**: seleccione **V2**.
    * **Ubicación**: Seleccione la ubicación de la factoría de datos. Solo las ubicaciones admitidas se muestran en la lista desplegable. Los almacenes de datos que las factorías de datos usan pueden estar en otras ubicaciones y regiones. Estos almacenes de datos incluyen Azure Data Lake Store, Azure Storage, Azure SQL Database, etc.

3. Seleccione **Crear**.
4. Una vez completada la creación, vaya a la factoría de datos. Verá la página principal de **Factoría de datos**, tal y como se muestra en la siguiente imagen:

    :::image type="content" source="./media/doc-common-process/data-factory-home-page.png" alt-text="Página principal de Azure Data Factory, con el icono de creación y supervisión.":::

   Seleccione el icono **Author & Monitor** (Creación y supervisión) para iniciar la aplicación de integración de datos en una pestaña independiente.

## <a name="load-data-into-azure-synapse-analytics"></a>Carga de datos en Azure Synapse Analytics

1. En la página **Introducción**, seleccione el icono **Copiar datos** para iniciar la herramienta para copiar datos.

2. En la página **Propiedades**, especifique **CopyFromSQLToSQLDW** en el campo **Nombre de la tarea** y seleccione **Siguiente**.

    ![Página de propiedades](./media/load-azure-sql-data-warehouse/copy-data-tool-properties-page.png)

3. En la página **Almacén de datos de origen**, realice los pasos siguientes:
    >[!TIP]
    >En este tutorial, usará *Autenticación de SQL* como tipo de autenticación para el almacén de datos de origen, pero puede elegir otros métodos de autenticación compatibles: *Entidad de servicio* e *Identidad administrada* si es necesario. Consulte las secciones correspondientes en [este artículo](./connector-azure-sql-database.md#linked-service-properties) para más información.
    >Para almacenar los secretos de los almacenes de datos de forma segura, también se recomienda usar Azure Key Vault. Consulte [este artículo](./store-credentials-in-key-vault.md) para obtener instrucciones detalladas.

    a. Haga clic en **+ Crear nueva conexión**.

    b. Seleccione **Azure SQL Database** en la galería y seleccione **Continuar**. Puede escribir "SQL" en el cuadro de búsqueda para filtrar los conectores.

    ![Seleccione Azure SQL DB](./media/load-azure-sql-data-warehouse/select-azure-sql-db-source.png)

    c. En la página **New Linked Service** (Nuevo servicio vinculado) seleccione el nombre del servidor y el de la base de datos en la lista desplegable y especifique el nombre de usuario y la contraseña. Haga clic en **Prueba de conexión** para validar la configuración y, después, seleccione **Crear**.

    ![Configurar Azure SQL DB](./media/load-azure-sql-data-warehouse/configure-azure-sql-db.png)

    d. Seleccione el servicio vinculado recién creado como origen y, a continuación, haga clic en **Siguiente**.

4. En la página **Select tables from which to copy the data or use a custom query** (Seleccionar tablas de donde copiar los datos o usar una consulta personalizada), escriba **SalesLT** para filtrar las tablas. Elija el cuadro **Seleccionar todo** para usar todas las tablas para la copia y, a continuación, seleccione **Siguiente**.

    ![Selección de las tablas de origen](./media/load-azure-sql-data-warehouse/select-source-tables.png)

5. En la página **Aplicar filtro**, especifique la configuración o seleccione **Siguiente**.

6. En la página **Almacén de datos de destino**, realice los pasos siguientes:
    >[!TIP]
    >En este tutorial, usará *Autenticación de SQL* como tipo de autenticación para el almacén de datos de destino, pero puede elegir otros métodos de autenticación compatibles: *Entidad de servicio* e *Identidad administrada* si es necesario. Consulte las secciones correspondientes en [este artículo](./connector-azure-sql-data-warehouse.md#linked-service-properties) para más información.
    >Para almacenar los secretos de los almacenes de datos de forma segura, también se recomienda usar Azure Key Vault. Consulte [este artículo](./store-credentials-in-key-vault.md) para obtener instrucciones detalladas.

    a. Haga clic en **+ Crear nueva conexión** para agregar una conexión.

    b. Seleccione **Azure Synapse Analytics** de la galería y seleccione **Continuar**.

    ![Selección de Azure Synapse Analytics](./media/load-azure-sql-data-warehouse/select-azure-sql-dw-sink.png)

    c. En la página **New Linked Service** (Nuevo servicio vinculado) seleccione el nombre del servidor y el de la base de datos en la lista desplegable y especifique el nombre de usuario y la contraseña. Haga clic en **Prueba de conexión** para validar la configuración y, después, seleccione **Crear**.

    ![Configuración de Azure Synapse Analytics](./media/load-azure-sql-data-warehouse/configure-azure-sql-dw.png)

    d. Seleccione el servicio vinculado recién creado como receptor y, a continuación, haga clic en **Siguiente**.

7. En la página **Table mapping** (Asignación de tabla), revise el contenido y seleccione **Next** (Siguiente). Se muestra una asignación de tabla inteligente. Las tablas de origen se asignan a las tablas de destino en función de los nombres de tabla. Si la tabla de origen no existe en el destino, Azure Data Factory crea una con el mismo nombre de manera predeterminada. También se puede asignar una tabla de origen a una tabla de destino existente.

   ![Página Table mapping (Asignación de tabla)](./media/load-azure-sql-data-warehouse/table-mapping.png)

8. En la página **Asignación de columnas**, revise el contenido y seleccione **Siguiente**. La asignación inteligente de tabla se basa en el nombre de las columnas. Si permite que Data Factory cree automáticamente las tablas, la conversión de tipos de datos puede producirse cuando haya incompatibilidades entre los almacenes de origen y de destino. Si hay una conversión de tipos de datos no compatibles entre las columnas de origen y de destino, verá un mensaje de error junto a la tabla correspondiente.

    ![Página de asignación de columnas](./media/load-azure-sql-data-warehouse/schema-mapping.png)

9. En la página **Settings** (Configuración), siga estos pasos:

    a. En la sección **Staging settings** (Configuración provisional), haga clic en **+ New** (+Nuevo) para crear un almacenamiento provisional. El almacenamiento se usa para almacenar provisionalmente los datos antes de cargarlos en Azure Synapse Analytics mediante PolyBase. Una vez que se completa la copia, los datos provisionales en Azure Blob Storage se limpian automáticamente.

    b. En la página **New Linked Service** (Nuevo servicio vinculado), seleccione su cuenta de almacenamiento y seleccione **Crear** para implementar el servicio vinculado.

    c. Anule la selección de la opción **Use type default** (Usar tipo predeterminado) y luego seleccione **Siguiente**.

    ![Configurar PolyBase](./media/load-azure-sql-data-warehouse/configure-polybase.png)

10. En la página **Resumen**, revise la configuración y seleccione **Siguiente**.

    ![Página de resumen](./media/load-azure-sql-data-warehouse/summary-page.png)

11. En la página **Deployment** (Implementación), seleccione **Monitor** (Supervisión) para supervisar la canalización (tarea). 
 
12. Observe que la pestaña **Monitor** (Supervisión) de la izquierda se selecciona automáticamente. Cuando la ejecución de la canalización finalice correctamente, seleccione el vínculo **CopyFromSQLToSQLDW** en la columna **PIPELINE NAME** para ver los detalles de la ejecución de actividad o volver a ejecutar la canalización.

    [![Supervisión de las ejecuciones de canalización](./media/load-azure-sql-data-warehouse/pipeline-monitoring.png)](./media/load-azure-sql-data-warehouse/pipeline-monitoring.png#lightbox)

13. Para volver a la vista de ejecuciones de canalización, seleccione el vínculo **Todas las ejecuciones de la canalización**. Seleccione **Refresh** (Actualizar) para actualizar la lista.

    ![Supervisión de las ejecuciones de actividad](./media/load-azure-sql-data-warehouse/activity-monitoring.png)

14. Para supervisar los detalles de la ejecución de cada actividad de copia, seleccione el vínculo **Detalles** (con un icono de gafas) en **ACTIVITY NAME** en la vista de ejecuciones de actividad. Puede supervisar detalles como el volumen de datos copiados desde el origen al receptor, el rendimiento de los datos, los pasos de ejecución con su duración correspondiente y las configuraciones que se utilizan.
    ![Supervisión de los detalles de la ejecución de actividad en primer lugar](./media/load-azure-sql-data-warehouse/monitor-activity-run-details-1.png)

    ![Supervisión de los detalles de la ejecución de actividad en segundo lugar](./media/load-azure-sql-data-warehouse/monitor-activity-run-details-2.png)

## <a name="next-steps"></a>Pasos siguientes

Continúe al artículo siguiente para obtener información sobre la compatibilidad de Azure Synapse Analytics:

> [!div class="nextstepaction"]
>[Conector de Azure Synapse Analytics](connector-azure-sql-data-warehouse.md)