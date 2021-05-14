---
title: Copia incremental de datos con Captura de datos modificados
description: En este tutorial, creará una canalización de Azure Data Factory que copia de forma incremental en Azure Storage los datos diferenciales de una tabla de una base de datos de Instancia administrada de Azure SQL.
ms.author: yexu
author: dearandyxu
ms.service: data-factory
ms.topic: tutorial
ms.date: 02/18/2021
ms.openlocfilehash: 6c7bfaf6bd6ca11593a986ca8d929faf063be720
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107905781"
---
# <a name="incrementally-load-data-from-azure-sql-managed-instance-to-azure-storage-using-change-data-capture-cdc"></a>Carga incremental de datos de Instancia administrada de Azure SQL a Azure Blob Storage mediante la captura de datos modificados (CDC)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este tutorial, creará una factoría de datos de Azure con una canalización que carga en una instancia de Azure Blob Storage los datos diferenciales según la información de **captura de datos modificados (CDC)** de la base de datos de Instancia administrada de Azure SQL de origen.  

En este tutorial, realizará los siguientes pasos:

> [!div class="checklist"]
> * Preparación del almacenamiento de datos de origen
> * Creación de una factoría de datos.
> * Cree servicios vinculados.
> * Creación de conjuntos de datos de origen y receptores.
> * Creación, depuración y ejecución de la canalización para comprobar los datos modificados
> * Modificación de los datos de la tabla de origen
> * Creación, ejecución y supervisión de la canalización de copia incremental completa

## <a name="overview"></a>Información general
La tecnología de captura de datos modificados compatible con los almacenes de datos, como Instancias administradas (MI) de Azure SQL y SQL Server, se puede usar para identificar los datos modificados.  En este tutorial se describe cómo usar Azure Data Factory con la tecnología de captura de datos modificados de SQL para cargar de forma incremental los datos diferenciales de Instancia administrada de SQL en Azure Blob Storage.  Para información más concreta sobre la tecnología de captura de datos modificados de SQL, consulte [Acerca de la captura de datos modificados (SQL Server)](/sql/relational-databases/track-changes/about-change-data-capture-sql-server).

## <a name="end-to-end-workflow"></a>Flujo de trabajo de un extremo a otro
Estos son los pasos del flujo de trabajo típico de un extremo a otro para cargar datos de forma incremental mediante la tecnología de captura de datos modificados.

> [!NOTE]
> Tanto Instancia administrada de Azure SQL como SQL Server admiten la tecnología de captura de datos modificados. En este tutorial se usa Instancia administrada de Azure SQL como almacén de datos de origen. También puede usar un servidor local de SQL Server.

## <a name="high-level-solution"></a>Solución de alto nivel
En este tutorial, creará una canalización que realiza las siguientes operaciones:  

   1. Crea una **actividad de búsqueda** para contar el número de registros modificados en la tabla CDC de SQL Database y la pasa a una actividad de condición IF.
   2. Crea una **condición If** para comprobar si hay registros modificados y, en caso afirmativo, invoca la actividad de copia.
   3. Crea una **actividad de copia** para copiar los datos insertados, actualizados o eliminados entre la tabla CDC y Azure Blob Storage.

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos
* **Instancia administrada de Azure SQL Database**. La base de datos se usa como almacén de datos de **origen**. Si no tiene una instancia administrada de Azure SQL Database, consulte los pasos del artículo [Creación de una instancia administrada de Azure SQL Database](../azure-sql/managed-instance/instance-create-quickstart.md) para crear una.
* **Cuenta de Azure Storage**. Blob Storage se usa como almacén de datos **receptor**. Si no tiene una cuenta de almacenamiento de Azure, consulte el artículo [Crear una cuenta de almacenamiento](../storage/common/storage-account-create.md) para ver los pasos para su creación. Cree un contenedor llamado **raw**. 

### <a name="create-a-data-source-table-in-azure-sql-database"></a>Creación de una tabla de origen de datos en Azure SQL Database

1. Inicie **SQL Server Management Studio** y conéctese al servidor de Instancias administradas de Azure SQL.
2. En el **Explorador de servidores**, haga clic con el botón derecho en la **base de datos** y elija la **Nueva consulta**.
3. Ejecute el siguiente comando SQL en su base de datos de Instancias administradas de Azure SQL para crear una tabla llamada `customers` como almacén de origen de datos.  

    ```sql
    create table customers 
    (
    customer_id int, 
    first_name varchar(50), 
    last_name varchar(50), 
    email varchar(100), 
    city varchar(50), CONSTRAINT "PK_Customers" PRIMARY KEY CLUSTERED ("customer_id") 
     );
    ```
4. Habilite el mecanismo **Captura de datos modificados** en la base de datos y la tabla de origen (customers) mediante la ejecución de la siguiente consulta SQL:

    > [!NOTE]
    > - Reemplace &lt;el nombre del esquema de origen&gt; por el esquema de Instancia administrada de Azure SQL que tiene la tabla customers.
    > - La captura de datos modificados no hace nada como parte de las transacciones que cambian la tabla de la que se realiza el seguimiento. En cambio, las operaciones de inserción, actualización y eliminación se escriben en el registro de transacciones. Los datos que se depositan en las tablas de cambios crecerán hasta llegar a ser incontrolables si no se reduce su número de forma periódica y sistemática. Para más información, consulte [Habilitar la captura de datos modificados para una base de datos](/sql/relational-databases/track-changes/enable-and-disable-change-data-capture-sql-server#enable-change-data-capture-for-a-database).

    ```sql
    EXEC sys.sp_cdc_enable_db 
    
    EXEC sys.sp_cdc_enable_table
    @source_schema = 'dbo',
    @source_name = 'customers', 
    @role_name = 'null',
    @supports_net_changes = 1
    ```
5. Inserte los datos en la tabla customers ejecutando el comando siguiente:

    ```sql
     insert into customers 
         (customer_id, first_name, last_name, email, city) 
     values 
         (1, 'Chevy', 'Leward', 'cleward0@mapy.cz', 'Reading'),
         (2, 'Sayre', 'Ateggart', 'sateggart1@nih.gov', 'Portsmouth'),
        (3, 'Nathalia', 'Seckom', 'nseckom2@blogger.com', 'Portsmouth');
    ```

    > [!NOTE]
    > Antes de que se habilite la captura de datos modificados, no se capturan cambios históricos en la tabla.

## <a name="create-a-data-factory"></a>Crear una factoría de datos

1. Inicie el explorador web **Microsoft Edge** o **Google Chrome**. Actualmente, la interfaz de usuario de Data Factory solo se admite en los exploradores web Microsoft Edge y Google Chrome.
1. En el menú de la izquierda, seleccione **Crear un recurso** > **Datos y análisis** > **Data Factory**:

   ![Selección de la factoría de datos en el panel Nuevo](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-azure-data-factory-menu.png)

2. En la página **New data factory** (Nueva factoría de datos), escriba **ADFTutorialDataFactory** en **Name** (Nombre).

     ![Página New data factory (Nueva factoría de datos)](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-azure-data-factory.png)

   El nombre de la instancia de Azure Data Factory debe ser **único de forma global**. Si recibe el siguiente error, cambie el nombre de la factoría de datos (por ejemplo, yournameADFTutorialDataFactory) e intente crearlo de nuevo. Consulte el artículo [Azure Data Factory: reglas de nomenclatura](naming-rules.md) para conocer las reglas de nomenclatura de los artefactos de Data Factory.

    *El nombre de factoría de datos "ADFTutorialDataFactory" no está disponible.*
3. Seleccione **V2** para la **versión**.
4. Seleccione la **suscripción** de Azure donde desea crear la factoría de datos.
5. Para el **grupo de recursos**, realice uno de los siguientes pasos:

   1. Seleccione en primer lugar **Usar existente** y después un grupo de recursos de la lista desplegable.
   2. Seleccione **Crear nuevo** y escriba el nombre de un grupo de recursos.   
         
    Para obtener más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](../azure-resource-manager/management/overview.md).  
5. Seleccione la **ubicación** de Data Factory. En la lista desplegable solo se muestran las ubicaciones que se admiten. Los almacenes de datos (Azure Storage, Azure SQL Database, etc.) y los procesos (HDInsight, etc.) que usa la factoría de datos pueden encontrarse en otras regiones.
6. Anule la selección de **Habilitar GIT**.     
7. Haga clic en **Crear**.
8. Una vez finalizada la implementación, haga clic en **Ir al recurso**.

   ![La captura de pantalla muestra un mensaje que indica que la implementación ha finalizado y una opción para ir a recurso.](./media/tutorial-incremental-copy-change-data-capture-feature-portal/data-factory-deploy-complete.png)
9. Una vez completada la creación, verá la página **Data Factory** tal como se muestra en la imagen.

   ![La captura de pantalla muestra la factoría de datos que ha implementado.](./media/tutorial-incremental-copy-change-data-capture-feature-portal/data-factory-home-page.png)
10. Haga clic en el icono **Author & Monitor** (Creación y supervisión) para iniciar la interfaz de usuario de Azure Data Factory en una pestaña independiente.
11. En la página de **introducción**, cambie a la pestaña **Edit** (Editar) del panel izquierdo tal como se muestra en la siguiente imagen:

    ![Botón Create pipeline (Crear canalización)](./media/tutorial-incremental-copy-change-data-capture-feature-portal/get-started-page.png)

## <a name="create-linked-services"></a>Crear servicios vinculados
Los servicios vinculados se crean en una factoría de datos para vincular los almacenes de datos y los servicios de proceso con la factoría de datos. En esta sección, creará servicios vinculados a la cuenta de Azure Storage e Instancia administrada de Azure SQL.

### <a name="create-azure-storage-linked-service"></a>Creación de un servicio vinculado de Azure Storage
En este paso, vincula su cuenta de Azure Storage a la factoría de datos.

1. Haga clic en **Connections** (Conexiones) y en **+ New** (+ Nuevo).

   ![Botón para nueva conexión](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-connection-button-storage.png)
2. En la ventana **New Linked Service** (Nuevo servicio vinculado), seleccione **Azure Blob Storage** y haga clic en **Continue** (Continuar).

   ![Seleccionar Azure Blob Storage](./media/tutorial-incremental-copy-change-data-capture-feature-portal/select-azure-storage.png)
3. En la ventana **New Linked Service** (Nuevo servicio vinculado), realice los pasos siguientes:

   1. Escriba **AzureStorageLinkedService** en **Name** (Nombre).
   2. Seleccione la cuenta de Azure Storage en **Storage account name** (Nombre de la cuenta de Storage).
   3. Haga clic en **Save**(Guardar).

   ![Configuración de la cuenta de Azure Storage](./media/tutorial-incremental-copy-change-data-capture-feature-portal/azure-storage-linked-service-settings.png)


### <a name="create-azure-sql-mi-database-linked-service"></a>Cree un servicio vinculado de base de datos de Instancia administrada de Azure SQL.
En este paso, vinculará la base de datos de Instancia administrada de Azure SQL a la factoría de datos.

> [!NOTE]
> Para aquellos que usan Instancia administrada de SQL, consulte [aquí](./connector-azure-sql-managed-instance.md#prerequisites) para información sobre el acceso a través de un punto de conexión público frente a privado. Si usa un punto de conexión privado, tendría que ejecutar esta canalización mediante un entorno de ejecución de integración autohospedado. Lo mismo se aplica a los que ejecutan SQL Server en el entorno local, en una máquina virtual o en escenarios de red virtual.

1. Haga clic en **Connections** (Conexiones) y en **+ New** (+ Nuevo).
2. En la ventana **New Linked Service** (Nuevo servicio vinculado), seleccione **Azure SQL Database Managed Instance** (Instancia administrada de Azure SQL Database) y haga clic en **Continue** (Continuar).
3. En la ventana **New Linked Service** (Nuevo servicio vinculado), realice los pasos siguientes:

   1. Escriba **AzureSqlMI1** en el campo **Name** (Nombre).
   2. Seleccione su servidor SQL en el campo **Server name** (Nombre del servidor).
   4. Seleccione la base de datos de SQL en el campo **Database name** (Nombre de la base de datos).
   5. Escriba el nombre del usuario en el campo **User name** (Nombre de usuario).
   6. Escriba la contraseña del usuario en el campo **Password** (Contraseña).
   7. Haga clic en **Test connection** (Prueba de conexión) para probar la conexión.
   8. Haga clic en **Save** (Guardar) para guardar el servicio vinculado.

   ![Configuración del servicio vinculado de Instancia administrada de Azure SQL Database](./media/tutorial-incremental-copy-change-data-capture-feature-portal/azure-sql-managed-instance-database-linked-service-settings.png)

## <a name="create-datasets"></a>Creación de conjuntos de datos
En este paso, creará conjuntos de datos para representar el origen y el destino de los datos.

### <a name="create-a-dataset-to-represent-source-data"></a>Creación de un conjunto de datos que represente datos de origen
En este paso, creará conjuntos de datos para representar el origen de datos.

1. En la vista de árbol, haga clic en el **signo + (más)** y en **Dataset** (Conjunto de datos).

   ![Menú New Dataset (Nuevo conjunto de datos)](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-dataset-menu.png)
2. Seleccione **Azure SQL Database Managed Instance** (Instancia administrada de Azure SQL Database) y haga clic en **Continue** (Continuar).

   ![Tipo de conjunto de datos de origen: Azure SQL Database](./media/tutorial-incremental-copy-change-data-capture-feature-portal/select-azure-sql-database.png)
   
3. En la pestaña **Set properties** (Establecer propiedades), establezca la información de nombre y conexión del conjunto de datos:
 
   1. Seleccione **AzureSqlMI1** en **Linked service** (Servicio vinculado).
   2. Seleccione **[dbo].[dbo_customers_CT]** en **Table name** (Nombre de la tabla).  Nota: Esta tabla se creó automáticamente cuando se habilitó CDC en la tabla customers. Los datos modificados nunca se consultan en esta tabla directamente, sino que se extraen a través de las [funciones CDC](/sql/relational-databases/system-functions/change-data-capture-functions-transact-sql).

   ![Conexión de origen](./media/tutorial-incremental-copy-change-data-capture-feature-portal/source-dataset-configuration.png)

### <a name="create-a-dataset-to-represent-data-copied-to-sink-data-store"></a>Creación de un conjunto de datos que represente los datos copiados en el almacén de datos receptor
En este paso, creará un conjunto de datos para representar los datos que se copian desde el almacén de datos de origen. Como parte de los requisitos previos, ha creado el contenedor de lago de datos en la instancia de Azure Blob Storage. Cree el contenedor si no existe (o) asígnele el nombre de uno existente. En este tutorial, el nombre del archivo de salida se genera dinámicamente mediante el desencadenador de hora, que se configurará más tarde.

1. En la vista de árbol, haga clic en el **signo + (más)** y en **Dataset** (Conjunto de datos).

   ![Menú New Dataset (Nuevo conjunto de datos)](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-dataset-menu.png)
2. Seleccione **Azure Blob Storage** y haga clic en **Continue** (Continuar).

   ![Tipo de conjunto de datos receptor: Azure Blob Storage](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-type.png)
3. Seleccione **DelimitedText** y haga clic en **Continue** (Continuar).

   ![Formato del conjunto de dato del receptor: DelimitedText](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-format.png)
4. En la pestaña **Set properties** (Establecer propiedades), establezca la información de nombre y conexión del conjunto de datos:

   1. Seleccione **AzureStorageLinkedService** en **Linked service** (Servicio vinculado).
   2. Escriba **raw** (sin procesar) para la parte de **container** (contenedor) de **filePath**.
   3. Habilite **First row as header** (Primera fila como encabezado).
   4. Haga clic en **Aceptar**.

   ![Conjunto de datos receptor: conexión](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-configuration.png)

## <a name="create-a-pipeline-to-copy-the-changed-data"></a>Creación de una canalización para copiar los datos modificados
En este paso, creará una canalización, que primero comprueba el número de registros modificados presentes en la tabla de cambios mediante una **actividad de búsqueda**. Una actividad de condición IF comprueba si el número de registros modificados es mayor que cero y ejecuta una **actividad de copia** para copiar los datos insertados, actualizados o eliminados de Azure SQL Database a Azure Blob Storage. Por último, se configura un desencadenador de ventana de saltos de tamaño constante y las horas de inicio y de finalización se pasarán a las actividades como parámetros de la ventana de inicio y de finalización. 

1. En la interfaz de usuario de Data Factory, cambie a la pestaña **Edit** (Editar). Haga clic en el **signo + (más)** en el panel izquierdo y en **Pipeline** (Canalización).

    ![Menú New pipeline (Nueva canalización)](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-pipeline-menu.png)
2. Verá una nueva pestaña para configurar la canalización. También verá la canalización en la vista de árbol. En la ventana **Properties** (Propiedades), cambie el nombre de la canalización a **IncrementalCopyPipeline**.

    ![Nombre de la canalización](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-name.png)
3. En el cuadro de herramientas **Activities** (Actividades), expanda **General** (General), arrastre la actividad **Lookup** (Búsqueda) y colóquela en la superficie del diseñador de canalizaciones. Establezca el nombre de la actividad en **GetChangeCount**. Esta actividad obtiene el número de registros de la tabla de cambios durante un período de tiempo determinado.

    ![Actividad de búsqueda: nombre](./media/tutorial-incremental-copy-change-data-capture-feature-portal/first-lookup-activity-name.png)
4. Cambie a la pestaña **Settings** (Configuración) de la ventana **Properties** (Propiedades):
   1. Especifique el nombre del conjunto de datos de Instancia administrada de SQL en el campo **Source Dataset** (Conjunto de datos de origen).
   2. Seleccione la opción de consulta y escriba lo siguiente en el cuadro de consulta:
    ```sql
    DECLARE  @from_lsn binary(10), @to_lsn binary(10);  
    SET @from_lsn =sys.fn_cdc_get_min_lsn('dbo_customers');  
    SET @to_lsn = sys.fn_cdc_map_time_to_lsn('largest less than or equal',  GETDATE());
    SELECT count(1) changecount FROM cdc.fn_cdc_get_all_changes_dbo_customers(@from_lsn, @to_lsn, 'all')
    ```
   3. Habilite **First row only** (Solo primera fila).

    ![Actividad de búsqueda: configuración](./media/tutorial-incremental-copy-change-data-capture-feature-portal/first-lookup-activity-settings.png)
5. Haga clic en el botón **Preview data** (Vista previa de datos) para asegurarse de que la actividad de búsqueda obtiene una salida válida.

    ![Actividad de búsqueda: vista previa](./media/tutorial-incremental-copy-change-data-capture-feature-portal/first-lookup-activity-preview.png)
6. Expanda **Iteration & conditionals** (Iteración y condicionales) en el cuadro de herramientas **Activities** (Actividades) y arrastre y coloque la actividad **If Condition** (Condición If) en la superficie del diseñador de canalizaciones. Establezca el nombre de la actividad en **HasChangedRows**. 

    ![Actividad de condición If: nombre](./media/tutorial-incremental-copy-change-data-capture-feature-portal/if-condition-activity-name.png)
7. Cambie a **Activities** (Actividades) en la ventana **Properties** (Propiedades):

   1. Escriba la siguiente **expresión**.
   
    ```adf
    @greater(int(activity('GetChangeCount').output.firstRow.changecount),0)
    ```

   2. Haga clic en el icono de lápiz para editar la condición True.

   ![Actividad If Condition (Condición If): configuración](./media/tutorial-incremental-copy-change-data-capture-feature-portal/if-condition-activity-setting.png)

   3. En el cuadro de herramientas **Activities** (Actividades), expanda **General** (General), arrastre la actividad **Wait** (Espera) y colóquela en la superficie del diseñador de canalizaciones. Esta es una actividad temporal para depurar la condición If y se modificará más adelante en el tutorial. 

   ![If Condition (Condición If) True: espera](./media/tutorial-incremental-copy-change-data-capture-feature-portal/if-condition-activity-wait.png)

   4. Haga clic en la ruta de navegación de IncrementalCopyPipeline para volver a la canalización principal.

8. Ejecute la canalización en modo **Debug** (Depurar) para comprobar que se ejecuta correctamente. 

   ![Canalización: depuración](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-debug.png)
9. A continuación, vuelva al paso de condición True y elimine la actividad **Wait** (Espera). En el cuadro de herramientas **Activities** (Actividades), expanda **Move & transform** (Mover y transformar), arrastre la actividad **Copy** (Copia) y colóquela en la superficie del diseñador de canalizaciones. Establezca el nombre de la actividad en **IncrementalCopyActivity**. 

   ![Actividad de copia: nombre](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-source-name.png)
10. Cambie a la pestaña **Source** (Origen) de la ventana **Properties** (Propiedades) y realice los pasos siguientes:

   1. Especifique el nombre del conjunto de datos de Instancia administrada de SQL en el campo **Source Dataset** (Conjunto de datos de origen). 
   2. Seleccione **Query** (Consulta) en **Use Query** (Usar consulta).
   3. En **Query** (Consulta), escriba lo siguiente.

      ```sql
      DECLARE @from_lsn binary(10), @to_lsn binary(10); 
      SET @from_lsn =sys.fn_cdc_get_min_lsn('dbo_customers'); 
      SET @to_lsn = sys.fn_cdc_map_time_to_lsn('largest less than or equal', GETDATE());
      SELECT * FROM cdc.fn_cdc_get_all_changes_dbo_customers(@from_lsn, @to_lsn, 'all')
      ```

   ![Actividad de copia: configuración del origen](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-source-settings.png)

11. Haga clic en Preview (Vista previa) para comprobar que la consulta devuelve las filas modificadas correctamente.

    ![La captura de pantalla muestra la vista previa para comprobar la consulta.](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-source-preview.png)
12. Cambie a la pestaña **Sink** (Receptor) y especifique el conjunto de datos de Azure Storage en el campo **Sink Dataset** (Conjunto de datos de receptor).

    ![La captura de pantalla muestra la pestaña de receptor.](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-sink-settings.png)
13. Haga clic de nuevo en el lienzo de la canalización principal y conecte la actividad **Lookup** (Búsqueda) a la actividad **If Condition** (Condición If) una a una. Arrastre el botón **verde** asociado a la actividad **Lookup** (Búsqueda) a la actividad **If Condition** (Condición If).

    ![Conexión de las actividades de búsqueda y de copia](./media/tutorial-incremental-copy-change-data-capture-feature-portal/connect-lookup-if.png)
14. Haga clic en **Validate** (Comprobar) en la barra de herramientas. Confirme que no haya errores de comprobación. Para cerrar la ventana **Pipeline Validation Report** (Informe de comprobación de la canalización), haga clic en **>>** .

    ![Botón Validate (Comprobar)](./media/tutorial-incremental-copy-change-data-capture-feature-portal/validate-button.png)
15. Haga clic en Debug (Depurar) para probar la canalización y comprobar que se genera un archivo en la ubicación de almacenamiento.

    ![Depuración de canalización incremental 2](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-debug-2.png)
16. Para publicar entidades (servicios vinculados, conjuntos de datos y canalizaciones) en el servicio Data Factory, haga clic en el botón **Publicar todo**. Espere hasta ver el mensaje **Publishing succeeded** (Publicación correcta).

    ![Botón Publicar](./media/tutorial-incremental-copy-change-data-capture-feature-portal/publish-button-2.png)    

### <a name="configure-the-tumbling-window-trigger-and-cdc-window-parameters"></a>Configuración del desencadenador de ventana de saltos de tamaño constante y de los parámetros de la ventana CDC 
En este paso, creará un desencadenador de ventana de saltos de tamaño constante para ejecutar el trabajo según una programación frecuente. Usará las variables del sistema WindowStart y WindowEnd del desencadenador de ventana de saltos de tamaño constante y las pasará como parámetros a la canalización que se usará en la consulta CDC.

1. Vaya a la pestaña **Parameters** (Parámetros) de la canalización **IncrementalCopyPipeline** y, con el botón **+ New** (+ Nuevo), agregue dos parámetros (**triggerStartTime** y **triggerEndTime**) a la canalización, que representan la hora de inicio y de finalización de la ventana de saltos de tamaño constante. Con fines de depuración, agregue valores predeterminados con el formato **AAAA-MM-DD HH24:MI:SS.FFF**, pero asegúrese de que triggerStartTime no sea anterior a la CDC que se habilita en la tabla o se producirá un error.

    ![Menú Trigger Now (Desencadenar ahora)](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-parameters.png)
2. Haga clic en la pestaña de configuración de la actividad **Lookup** (Búsqueda) y configure la consulta para usar los parámetros de inicio y de finalización. Copie lo siguiente en la consulta:
    ```sql
    @concat('DECLARE @begin_time datetime, @end_time datetime, @from_lsn binary(10), @to_lsn binary(10); 
    SET @begin_time = ''',pipeline().parameters.triggerStartTime,''';
    SET @end_time = ''',pipeline().parameters.triggerEndTime,''';
    SET @from_lsn = sys.fn_cdc_map_time_to_lsn(''smallest greater than or equal'', @begin_time);
    SET @to_lsn = sys.fn_cdc_map_time_to_lsn(''largest less than'', @end_time);
    SELECT count(1) changecount FROM cdc.fn_cdc_get_all_changes_dbo_customers(@from_lsn, @to_lsn, ''all'')')
    ```

3. Vaya a la actividad **Copy** (Copia) en el caso True de la actividad **If Condition** (Condición if) y haga clic en la pestaña **Source** (Origen). Copie lo siguiente en la consulta:
    ```sql
    @concat('DECLARE @begin_time datetime, @end_time datetime, @from_lsn binary(10), @to_lsn binary(10); 
    SET @begin_time = ''',pipeline().parameters.triggerStartTime,''';
    SET @end_time = ''',pipeline().parameters.triggerEndTime,''';
    SET @from_lsn = sys.fn_cdc_map_time_to_lsn(''smallest greater than or equal'', @begin_time);
    SET @to_lsn = sys.fn_cdc_map_time_to_lsn(''largest less than'', @end_time);
    SELECT * FROM cdc.fn_cdc_get_all_changes_dbo_customers(@from_lsn, @to_lsn, ''all'')')
    ```
4. Haga clic en la pestaña **Sink** (Receptor) de la actividad **Copy** (Copia) y haga clic en **Open** (Abrir) para editar los parámetros del conjunto de datos. Haga clic en la pestaña **Parameters** (Parámetros) y agregue un nuevo parámetro denominado **triggerStart**    

    ![La captura de pantalla muestra cómo agregar un nuevo parámetro a la pestaña de parámetros.](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-configuration-2.png)
5. A continuación, configure las propiedades del conjunto de datos para almacenar los datos en el subdirectorio **clientes/incremental** con particiones basadas en fechas.
   1. Haga clic en la pestaña **Connection** (Conexión) de las propiedades del conjunto de datos y agregue contenido dinámico en las secciones **Directory** (Directorio) y **File** (Archivo). 
   2. Escriba la siguiente expresión en la sección **Directory** (Directorio); para ello, haga clic en el vínculo de contenido dinámico en el cuadro de texto:
    
    ```sql
    @concat('customers/incremental/',formatDateTime(dataset().triggerStart,'yyyy/MM/dd'))
    ```
   3. Escriba la siguiente expresión en la sección **File** (Archivo). Se crean nombres de archivo basados en la fecha y la hora de inicio del desencadenador, con el sufijo de la extensión csv:
    
    ```sql
    @concat(formatDateTime(dataset().triggerStart,'yyyyMMddHHmmssfff'),'.csv')
    ```
    ![Configuración del conjunto de datos de receptor: 3](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-configuration-3.png)

   4. Vuelva a la configuración de **Sink** (Receptor) de la actividad **Copy** (Copia) haciendo clic en la pestaña **IncrementalCopyPipeline**. 
   5. Expanda las propiedades del conjunto de datos y escriba contenido dinámico en el valor del parámetro triggerStart con la siguiente expresión:
     ```sql
     @pipeline().parameters.triggerStartTime
     ```
    ![Configuración del conjunto de datos de receptor 4](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-configuration-4.png)

6. Haga clic en Debug (Depurar) para probar la canalización y asegurarse de que la estructura de carpetas y el archivo de salida se generan según lo previsto. Descargue y abra el archivo para comprobar el contenido. 

    ![Depuración de copia incremental 3](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-debug-3.png)
7. Asegúrese de que los parámetros se inserten en la consulta, para lo cual debe revisar los parámetros de entrada de la ejecución de canalización.

    ![Depuración de copia incremental 4](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-debug-4.png)
8. Para publicar entidades (servicios vinculados, conjuntos de datos y canalizaciones) en el servicio Data Factory, haga clic en el botón **Publicar todo**. Espere hasta ver el mensaje **Publishing succeeded** (Publicación correcta).
9. Por último, configure un desencadenador de ventana de saltos de tamaño constante para ejecutar la canalización a intervalos regulares y establecer los parámetros de hora de inicio y de finalización. 
   1. Haga clic en el botón **Add trigger** (Agregar desencadenador) y seleccione **New/Edit** (Nuevo/Editar).

   ![Agregar nuevo desencadenador](./media/tutorial-incremental-copy-change-data-capture-feature-portal/add-trigger.png)

   2. Escriba un nombre de desencadenador y especifique una hora de inicio, que es igual a la hora de finalización de la ventana de depuración anterior.

   ![Desencadenador de ventana de saltos de tamaño constante](./media/tutorial-incremental-copy-change-data-capture-feature-portal/tumbling-window-trigger.png)

   3. En la siguiente pantalla, especifique los siguientes valores para los parámetros de inicio y de finalización, respectivamente.
    ```sql
    @formatDateTime(trigger().outputs.windowStartTime,'yyyy-MM-dd HH:mm:ss.fff')
    @formatDateTime(trigger().outputs.windowEndTime,'yyyy-MM-dd HH:mm:ss.fff')
    ```

   ![Desencadenador de ventana de saltos de tamaño constante 2](./media/tutorial-incremental-copy-change-data-capture-feature-portal/tumbling-window-trigger-2.png)

> [!NOTE]
> Tenga en cuenta que el desencadenador solo se ejecuta una vez que se ha publicado. Además, el comportamiento esperado de la ventana de saltos de tamaño constante es ejecutar todos los intervalos históricos desde la fecha de inicio hasta ahora. Puede encontrar más información sobre los desencadenadores de ventana de saltos de tamaño constante [aquí](./how-to-create-tumbling-window-trigger.md). 
  
10. Con **SQL Server Management Studio** realice cambios adicionales en la tabla customers mediante la ejecución del siguiente código SQL:
    ```sql
    insert into customers (customer_id, first_name, last_name, email, city) values (4, 'Farlie', 'Hadigate', 'fhadigate3@zdnet.com', 'Reading');
    insert into customers (customer_id, first_name, last_name, email, city) values (5, 'Anet', 'MacColm', 'amaccolm4@yellowbook.com', 'Portsmouth');
    insert into customers (customer_id, first_name, last_name, email, city) values (6, 'Elonore', 'Bearham', 'ebearham5@ebay.co.uk', 'Portsmouth');
    update customers set first_name='Elon' where customer_id=6;
    delete from customers where customer_id=5;
    ```
11. Haga clic en el botón **Publicar todo**. Espere hasta ver el mensaje **Publishing succeeded** (Publicación correcta).  
12. Al cabo de unos minutos, se habrá desencadenado la canalización y se habrá cargado un nuevo archivo en Azure Storage.


### <a name="monitor-the-incremental-copy-pipeline"></a>Supervisión de la canalización de la copia incremental
1. Haga clic en la pestaña **Monitor** (Supervisar) de la izquierda. Verá la ejecución de la canalización en la lista y su estado. Haga clic en **Refresh** (Actualizar) para actualizar la lista. Mantenga el puntero cerca del nombre de la canalización para acceder a la acción Rerun (Volver a ejecutar) y al informe de consumo.

    ![Ejecuciones de la canalización](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-pipeline-runs.png)
2. Para ver las ejecuciones de actividad asociadas a la ejecución de la canalización, haga clic en el nombre de la canalización. Si se han detectado datos modificados, habrá tres actividades, incluida la actividad de copia; de lo contrario, solo habrá dos entradas en la lista. Para volver a la vista Pipeline Runs (Ejecuciones de canalización), haga clic en el vínculo **All Pipelines** (Todas las canalizaciones) de la parte superior.

    ![Ejecuciones de actividad](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-activity-runs.png)


### <a name="review-the-results"></a>Revisión del resultado
Verá el segundo archivo `customers/incremental/YYYY/MM/DD` en la carpeta `raw` del contenedor.

![Archivo de salida de la copia incremental](media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-run.png)
 

## <a name="next-steps"></a>Pasos siguientes
Pase al tutorial siguiente para obtener información acerca de cómo copiar archivos nuevos y modificados solo según el valor de LastModifiedDate:

> [!div class="nextstepaction"]
>[Copia de archivos nuevos por LastModifiedDate](tutorial-incremental-copy-lastmodified-copy-data-tool.md)
