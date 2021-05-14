---
title: Actividades de bifurcación y encadenamiento en una canalización mediante Azure Portal
description: Obtenga información sobre cómo controlar el flujo de datos en la canalización de Azure Data Factory mediante Azure Portal.
author: ssabat
ms.author: susabat
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: tutorial
ms.custom: seo-lt-2019; seo-dt-2019
ms.date: 01/11/2018
ms.openlocfilehash: a1e30bb59d81e949720560f2ce5da796a2c7fbe2
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107905349"
---
# <a name="branching-and-chaining-activities-in-an-azure-data-factory-pipeline-using-the-azure-portal"></a>Bifurcación y encadenamiento de actividades en una canalización de Azure Data Factory mediante Azure Portal

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este tutorial, creará una canalización de Data Factory que muestra algunas de las características del flujo de control. Esta canalización realiza una copia simple de un contenedor en Azure Blob Storage a otro contenedor de la misma cuenta de almacenamiento. Si la actividad de copia se realiza correctamente, la canalización envía los detalles de la operación de copia correcta (por ejemplo, la cantidad de datos escritos) en un correo electrónico de operación correcta. Si se produce un error en la actividad de copia, la canalización envía los detalles del error de copia (por ejemplo, el mensaje de error) en un correo electrónico de operación incorrecta. A lo largo del tutorial, verá cómo pasar parámetros.

Información general del escenario: ![En el diagrama se muestra Azure Blob Storage, que es el destino de una copia que, si se ejecuta correctamente, envía un correo electrónico con los detalles o, en caso de error, envía un correo electrónico con los detalles del error.](media/tutorial-control-flow-portal/overview.png)

En este tutorial, realizará los siguientes pasos:

> [!div class="checklist"]
> * Creación de una factoría de datos.
> * Creación de un servicio vinculado de Azure Storage
> * Creación de un conjunto de datos del blob de Azure
> * Creación de una canalización que contenga una actividad de copia y una actividad web
> * Envío de los resultados de las actividades en actividades subsiguientes
> * Uso de las variables del sistema y del paso de parámetros
> * Inicio de la ejecución de una canalización
> * Supervisión de las ejecuciones de canalización y actividad

En este tutorial se usa Azure Portal. Puede usar otros mecanismos para interactuar con Azure Data Factory. Consulte "Inicios rápidos" en la tabla de contenido.

## <a name="prerequisites"></a>Requisitos previos

* **Suscripción de Azure**. Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.
* **Cuenta de Azure Storage**. Blob Storage se puede usar como almacén de datos de **origen**. Si no tiene una cuenta de almacenamiento de Azure, consulte el artículo [Crear una cuenta de almacenamiento](../storage/common/storage-account-create.md) para ver los pasos para su creación.
* **Azure SQL Database**. La base de datos se puede usar como almacén de datos **receptor**. Si no tiene ninguna base de datos en Azure SQL Database, consulte el artículo [Creación de una base de datos en Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md) para ver los pasos y crear una.

### <a name="create-blob-table"></a>Creación de la tabla de blobs

1. Inicie el Bloc de notas. Copie el texto siguiente y guárdelo como archivo **input.txt** en el disco.

    ```
    John,Doe
    Jane,Doe
    ```
2. Use herramientas como el [Explorador de Azure Storage](https://storageexplorer.com/) para realizar los pasos siguientes:
    1. Cree el contenedor **adfv2branch**.
    2. Cree la carpeta de **entrada** en el contenedor **adfv2branch**.
    3. Cargue el archivo **input.txt** en el contenedor.

## <a name="create-email-workflow-endpoints"></a>Creación de puntos de conexión de flujo de trabajo del correo electrónico
Para desencadenar el envío de un correo electrónico de la canalización, defina el flujo de trabajo con [Logic Apps](../logic-apps/logic-apps-overview.md). Para obtener más información acerca de cómo crear un flujo de trabajo de una aplicación lógica, consulte [Cómo crear una aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

### <a name="success-email-workflow"></a>Flujo de trabajo del correo electrónico de operación correcta
Cree un flujo de trabajo de aplicación lógica denominado `CopySuccessEmail`. Defina el desencadenador del flujo de trabajo como `When an HTTP request is received` y agregue una acción de `Office 365 Outlook – Send an email`.

![Flujo de trabajo del correo electrónico de operación correcta](media/tutorial-control-flow-portal/success-email-workflow.png)

Para el desencadenador de la solicitud, rellene `Request Body JSON Schema` con el JSON siguiente:

```json
{
    "properties": {
        "dataFactoryName": {
            "type": "string"
        },
        "message": {
            "type": "string"
        },
        "pipelineName": {
            "type": "string"
        },
        "receiver": {
            "type": "string"
        }
    },
    "type": "object"
}
```

La solicitud del Diseñador de aplicación lógica debe parecerse a la imagen siguiente:

![Diseñador de aplicación lógica: solicitud](media/tutorial-control-flow-portal/logic-app-designer-request.png)

Para la acción **Enviar correo electrónico**, personalice el formato del correo electrónico. Para ello, use las propiedades que se pasan en el esquema JSON del cuerpo de solicitud. Este es un ejemplo:

![Diseñador de aplicación lógica: acción de envío de correo electrónico](media/tutorial-control-flow-portal/send-email-action-2.png)

Guarde el flujo de trabajo. Tome nota de la URL de solicitud POST HTTP para el flujo de trabajo del correo electrónico de operación correcta:

```
//Success Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000
```

### <a name="fail-email-workflow"></a>Flujo de trabajo del correo electrónico de operación incorrecta
Siga los mismos pasos para crear otro flujo de trabajo de Logic Apps de **CopyFailEmail**. En el desencadenador de solicitudes, `Request Body JSON schema` es el mismo. Cambie el formato del correo electrónico, por ejemplo, la parte `Subject`, para adaptarlo para que sea un correo electrónico de operación incorrecta. Este es un ejemplo:

![Diseñador de aplicación lógica: flujo de trabajo del correo electrónico de operación incorrecta](media/tutorial-control-flow-portal/fail-email-workflow-2.png)

Guarde el flujo de trabajo. Tome nota de la URL de solicitud POST HTTP para el flujo de trabajo del correo electrónico de operación incorrecta:

```
//Fail Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000
```

Ahora debería tener dos URL de flujo de trabajo:

```
//Success Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000

//Fail Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000
```

## <a name="create-a-data-factory"></a>Crear una factoría de datos

1. Inicie el explorador web **Microsoft Edge** o **Google Chrome**. Actualmente, la interfaz de usuario de Data Factory solo se admite en los exploradores web Microsoft Edge y Google Chrome.
1. En el menú de la izquierda, seleccione **Crear un recurso** > **Datos y análisis** > **Data Factory**:

   ![Selección de la factoría de datos en el panel Nuevo](./media/quickstart-create-data-factory-portal/new-azure-data-factory-menu.png)

2. En la página **New data factory** (Nueva factoría de datos), escriba **ADFTutorialDataFactory** en **Name** (Nombre).

     ![Página New data factory (Nueva factoría de datos)](./media/tutorial-control-flow-portal/new-azure-data-factory.png)

   El nombre de la instancia de Azure Data Factory debe ser **único de forma global**. Si recibe el siguiente error, cambie el nombre de la factoría de datos (por ejemplo, yournameADFTutorialDataFactory) e intente crearlo de nuevo. Consulte el artículo [Azure Data Factory: reglas de nomenclatura](naming-rules.md) para conocer las reglas de nomenclatura de los artefactos de Data Factory.

   *El nombre "ADFTutorialDataFactory" de factoría de datos no está disponible.*

3. Seleccione la **suscripción** de Azure donde desea crear la factoría de datos.
4. Para el **grupo de recursos**, realice uno de los siguientes pasos:

      - Seleccione en primer lugar **Usar existente** y después un grupo de recursos de la lista desplegable.
      - Seleccione **Crear nuevo** y escriba el nombre de un grupo de recursos.   
         
        Para obtener más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](../azure-resource-manager/management/overview.md).  
4. Seleccione **V2** para la **versión**.
5. Seleccione la **ubicación** de Data Factory. En la lista desplegable solo se muestran las ubicaciones que se admiten. Los almacenes de datos (Azure Storage, Azure SQL Database, etc.) y los procesos (HDInsight, etc.) que usa la factoría de datos pueden encontrarse en otras regiones.
6. Seleccione **Anclar al panel**.     
7. Haga clic en **Crear**.      
8. En el panel, verá el icono siguiente con el estado: **Deploying data factory** (Implementación de la factoría de datos).

    ![icono implementando factoría de datos](media/tutorial-control-flow-portal/deploying-data-factory.png)
9. Una vez completada la creación, verá la página **Data Factory** tal como se muestra en la imagen.

   ![Página principal Factoría de datos](./media/tutorial-control-flow-portal/data-factory-home-page.png)
10. Haga clic en el icono **Author & Monitor** (Creación y supervisión) para iniciar la interfaz de usuario de Azure Data Factory en una pestaña independiente.


## <a name="create-a-pipeline"></a>Crear una canalización
En este paso se crea una canalización con una actividad de copia y dos actividades web. Use las siguientes características para crear la canalización:

- Los parámetros de acceso de los conjuntos de datos de la canalización.
- La actividad web para invocar flujos de trabajo de aplicaciones lógicas para enviar mensajes de correo electrónico de operación correcta/incorrecta.
- Conexión de una actividad con otra (con operaciones correctas e incorrectas)
- Uso de la salida de una actividad como la entrada para la actividad subsiguiente

1. En la página de **introducción** de la interfaz de usuario de Data Factory, haga clic en el icono **Create pipeline** (Crear canalización).  

   ![Página de introducción](./media/tutorial-control-flow-portal/get-started-page.png)
3. En la ventana de propiedades de la canalización, cambie a la pestaña **Parameters** (Parámetros) y a use el botón **New** (Nuevo) para agregar los siguientes tres parámetros de tipo String: sourceBlobContainer, sinkBlobContainer y receiver.

    - **sourceBlobContainer**: parámetro de la canalización que consume el conjunto de datos del blob de origen.
    - **sinkBlobContainer**: parámetro de la canalización que consume el conjunto de datos del blob receptor.
    - **receiver**: parámetro que usan las dos actividades web de la canalización que envían correos electrónicos de operación correcta o incorrecta al receptor cuya dirección de correo electrónico especifica este parámetro.

   ![Menú New pipeline (Nueva canalización)](./media/tutorial-control-flow-portal/pipeline-parameters.png)
4. En el cuadro de herramientas **Activities** (Actividades), expanda **Data Flow** (Flujo de datos), arrastre la actividad **Copy** (Copiar) y colóquela en la superficie del diseñador de canalizaciones.

   ![Arrastrar y colocar la actividad de copia](./media/tutorial-control-flow-portal/drag-drop-copy-activity.png)
5. En la ventana **Properties** (Propiedades) de la actividad **Copy** (Copia) de la parte inferior, cambie a la pestaña **Source** (origen) y haga clic en **+ New** (+ Nuevo). En este paso se crea un conjunto de datos de origen para la actividad de copia.

   ![Captura de pantalla que muestra cómo crear un conjunto de datos de origen para la actividad de copia.](./media/tutorial-control-flow-portal/new-source-dataset-button.png)
6. En la ventana **New Dataset** (Nuevo conjunto de datos), seleccione **Azure Blob Storage** y haga clic en **Finish** (Finalizar).

   ![Seleccionar Azure Blob Storage](./media/tutorial-control-flow-portal/select-azure-blob-storage.png)
7. Verá una nueva **pestaña** titulada **AzureBlob1**. Cámbiele el nombre al conjunto de datos a **SourceBlobDataset**.

   ![Configuración general del conjunto de datos](./media/tutorial-control-flow-portal/dataset-general-page.png)
8. Cambie a la pestaña **Connection** (Conexión) de la ventana **Properties** (Propiedades) y haga clic en New (Nuevo) en **Linked service** (Servicio vinculado). En este paso se crea un servicio vinculado para vincular la cuenta de Azure Storage con la factoría de datos.

   ![Conexión del conjunto de datos: nuevo servicio vinculado](./media/tutorial-control-flow-portal/dataset-connection-new-button.png)
9. En la ventana **New Linked Service** (Nuevo servicio vinculado), realice los pasos siguientes:

    1. Escriba **AzureStorageLinkedService** en **Name** (Nombre).
    2. Seleccione la cuenta de Azure Storage en **Storage account name** (Nombre de la cuenta de Storage).
    3. Haga clic en **Save**(Guardar).

   ![Nuevo servicio vinculado de Azure Storage](./media/tutorial-control-flow-portal/new-azure-storage-linked-service.png)
12. Escriba `@pipeline().parameters.sourceBlobContainer` para la carpeta y `emp.txt`, para el nombre de archivo. El parámetro de canalización sourceBlobContainer se usa para establecer la ruta de acceso de carpeta para el conjunto de datos.

    ![Configuración del conjunto de datos de origen](./media/tutorial-control-flow-portal/source-dataset-settings.png)

13. Cambie a la pestaña **Pipeline** (Canalización) (o) haga clic en la canalización en la vista de árbol. Confirme que **SourceBlobDataset** está seleccionado en **Source Dataset** (Conjunto de datos de origen).
      
   ![Conjunto de datos de origen](./media/tutorial-control-flow-portal/pipeline-source-dataset-selected.png)

13. En la ventana de propiedades, cambie a la pestaña **Sink** (Receptor) y haga clic en **+ New** (+ Nuevo) en **Sink Dataset** (Conjunto de datos receptor). En este paso se crea un conjunto de datos receptor para la actividad de copia, de manera similar a la creación del conjunto de datos de origen.

    ![Botón New sink dataset (Nuevo conjunto de datos receptor)](./media/tutorial-control-flow-portal/new-sink-dataset-button.png)
14. En la ventana **New Dataset** (Nuevo conjunto de datos), seleccione **Azure Blob Storage** y haga clic en **Finish** (Finalizar).
15. En la página de configuración **General** (General) del conjunto de datos, escriba **SinkBlobDataset** en **Name** (Nombre).
16. Cambie a la pestaña **Connection** (Conexión) y realice los pasos siguientes:

    1. Seleccione **AzureStorageLinkedService** como **Linked service** (Servicio vinculado).
    2. Escriba `@pipeline().parameters.sinkBlobContainer` como carpeta.
    1. Escriba `@CONCAT(pipeline().RunId, '.txt')` como nombre de archivo. La expresión usa el identificador de la ejecución de canalización actual del nombre de archivo. Para la lista de las expresiones y variables del sistema admitidas, consulte las [variables del sistema](control-flow-system-variables.md) y el [lenguaje de expresiones](control-flow-expression-language-functions.md).

        ![Configuración del conjunto de datos receptor](./media/tutorial-control-flow-portal/sink-dataset-settings.png)
17. Cambie a la pestaña de la **canalización** en la parte superior. En el cuadro de herramientas **Activities** (Actividades), expanda **General** (General), arrastre la actividad **web** y colóquela en la superficie del diseñador de canalizaciones. Establezca el nombre de la actividad en **SendSuccessEmailActivity**. La actividad web permite una llamada a cualquier punto de conexión de REST. Para más información sobre la actividad, consulte el artículo [Web Activity](control-flow-web-activity.md) (Actividad web). Esta canalización usa una actividad web para llamar al flujo de trabajo de correo electrónico de Logic Apps.

    ![Arrastrado y colocación de la primera actividad web](./media/tutorial-control-flow-portal/success-web-activity-general.png)
18. Cambie a la pestaña **Settings** (Configuración) en la pestaña **General** (General) y realice los pasos siguientes:
    1. Como **dirección URL**, especifique la dirección URL del flujo de trabajo de aplicaciones lógicas que envía el correo electrónico de operación correcta.  
    2. Seleccione **POST** como **Method** (Método).
    3. Haga clic en el vínculo **+ Add header** (+ Agregar encabezado) de la sección **Headers** (Encabezados).
    4. Agregue un encabezado **Content-Type** y establézcalo en **application/json**.
    5. Especifique el siguiente JSON para **Body** (Cuerpo).

        ```json
        {
            "message": "@{activity('Copy1').output.dataWritten}",
            "dataFactoryName": "@{pipeline().DataFactory}",
            "pipelineName": "@{pipeline().Pipeline}",
            "receiver": "@pipeline().parameters.receiver"
        }
        ```
        El cuerpo del mensaje contiene las siguientes propiedades:

       - Message, que pasa el valor `@{activity('Copy1').output.dataWritten`. Tiene acceso a una propiedad de la actividad de copia anterior y pasa el valor de dataWritten. En caso de error, pasa la salida de error en lugar de `@{activity('CopyBlobtoBlob').error.message`.
       - Data Factory Name, que pasa el valor `@{pipeline().DataFactory}`. Se trata de una variable del sistema, lo que le permite obtener acceso al nombre de la factoría de datos correspondiente. Para obtener una lista de las variables del sistema, vea el artículo [System Variables](control-flow-system-variables.md) (Variables del sistema).
       - Pipeline Name, que pasa el valor `@{pipeline().Pipeline}`. También es una variable del sistema, lo que le permite obtener acceso al nombre de canalización correspondiente.
       - Receiver, que pasa el valor "\@pipeline().parameters.receiver"). y accede a los parámetros de la canalización.

         ![Configuración de la primera actividad web](./media/tutorial-control-flow-portal/web-activity1-settings.png)         
19. Conecte la actividad **Copy** (Copiar) a la actividad **Web**; para ello, arrastre el botón verde situado junto a la actividad de copia y colóquelo en la actividad web.

    ![Conexión de la actividad de copia con la primera actividad web](./media/tutorial-control-flow-portal/connect-copy-web-activity1.png)
20. Arrastre la actividad **Web** del cuadro de herramientas de actividades y colóquela en la superficie del diseñador de canalizaciones; después, establezca el **nombre** en **SendFailureEmailActivity**.

    ![Nombre de la segunda actividad web](./media/tutorial-control-flow-portal/web-activity2-name.png)
21. Cambie a la pestaña **Settings** (Configuración) y realice los pasos siguientes:

    1. Como **dirección URL**, especifique la dirección URL del flujo de trabajo de aplicaciones lógicas que envía el correo electrónico de operación incorrecta.  
    2. Seleccione **POST** como **Method** (Método).
    3. Haga clic en el vínculo **+ Add header** (+ Agregar encabezado) de la sección **Headers** (Encabezados).
    4. Agregue un encabezado **Content-Type** y establézcalo en **application/json**.
    5. Especifique el siguiente JSON para **Body** (Cuerpo).

        ```json
        {
            "message": "@{activity('Copy1').error.message}",
            "dataFactoryName": "@{pipeline().DataFactory}",
            "pipelineName": "@{pipeline().Pipeline}",
            "receiver": "@pipeline().parameters.receiver"
        }
        ```

        ![Configuración de la segunda actividad web](./media/tutorial-control-flow-portal/web-activity2-settings.png)         
22. Seleccione la actividad **Copy** (Copiar) en el diseñador de canalizaciones, haga clic en el botón **+->** y seleccione **Error**.  

    ![Captura de pantalla que muestra cómo seleccionar un error en la actividad de copia en el diseñador de canalizaciones.](./media/tutorial-control-flow-portal/select-copy-failure-link.png)
23. Arrastre el botón **rojo** situado junto a la actividad de copia a la segunda actividad web **SendFailureEmailActivity**. Puede mover las actividades de forma que la canalización tenga un aspecto similar al de la siguiente imagen:

    ![Canalización completa con todas las actividades](./media/tutorial-control-flow-portal/full-pipeline.png)
24. Para comprobar la canalización, haga clic en el botón **Validate** (Comprobar) en la barra de herramientas. Haga clic en el botón **>>** para cerrar la ventana **Pipeline Validation Output** (Salida de comprobación de canalización).

    ![Comprobar la canalización](./media/tutorial-control-flow-portal/validate-pipeline.png)
24. Para publicar las entidades (conjuntos de datos, canalizaciones, etc.) en el servicio Data Factory, seleccione **Publish All** (Publicar todo). Espere a que aparezca el mensaje **Successfully published** (Publicado correctamente).

    ![Publicar](./media/tutorial-control-flow-portal/publish-button.png)

## <a name="trigger-a-pipeline-run-that-succeeds"></a>Desencadenamiento de una ejecución de la canalización que se realiza correctamente
1. Para **desencadenar** una ejecución de canalización, haga clic en **Trigger** (Desencadenar) en la barra de herramientas y en **Trigger Now** (Desencadenar ahora).

    ![Desencadenamiento de una ejecución de la canalización](./media/tutorial-control-flow-portal/trigger-now-menu.png)
2. En la ventana **Pipeline Run** (Ejecución de canalización), lleve a cabo los pasos siguientes:

    1. Escriba **adftutorial/adfv2branch/input** como parámetro **sourceBlobContainer**.
    2. Escriba **adftutorial/adfv2branch/output** como parámetro **sinkBlobContainer**.
    3. Escriba una **dirección de correo electrónico** en **receiver**.
    4. Haga clic en **Finish** (Finalizar).

        ![Parámetros de ejecución de canalización](./media/tutorial-control-flow-portal/pipeline-run-parameters.png)

## <a name="monitor-the-successful-pipeline-run"></a>Supervisión de la correcta ejecución de la canalización

1. Para supervisar la ejecución de la canalización, cambie a la pestaña **Monitor** (Supervisar) de la izquierda. Verá que la ejecución de la canalización que desencadenó manualmente. Use el botón **Refresh** (Actualizar) para actualizar la lista.

    ![Ejecución de canalización correcta](./media/tutorial-control-flow-portal/monitor-success-pipeline-run.png)
2. Para ver las **ejecuciones de actividad** asociadas con la de esta canalización, haga clic en el primer vínculo de la columna **Actions** (Acciones). Para volver a la vista anterior, haga clic en **Pipelines** (Canalizaciones) de la parte superior. Use el botón **Refresh** (Actualizar) para actualizar la lista.

    ![Captura de pantalla que muestra cómo ver la lista de ejecuciones de actividad.](./media/tutorial-control-flow-portal/activity-runs-success.png)

## <a name="trigger-a-pipeline-run-that-fails"></a>Desencadenamiento de una ejecución de la canalización que se realiza incorrectamente
1. Cambie a la pestaña **Edit** (Editar) de la izquierda.
2. Para **desencadenar** una ejecución de canalización, haga clic en **Trigger** (Desencadenar) en la barra de herramientas y en **Trigger Now** (Desencadenar ahora).
3. En la ventana **Pipeline Run** (Ejecución de canalización), lleve a cabo los pasos siguientes:

    1. Escriba **adftutorial/dummy/input** como parámetro **sourceBlobContainer**. Asegúrese de que la carpeta ficticia no existe en el contenedor adftutorial.
    2. Escriba **adftutorial/dummy/output** en el parámetro **sinkBlobContainer**.
    3. Escriba una **dirección de correo electrónico** en **receiver**.
    4. Haga clic en **Finalizar**

## <a name="monitor-the-failed-pipeline-run"></a>Supervisión de la ejecución de canalización incorrecta

1. Para supervisar la ejecución de la canalización, cambie a la pestaña **Monitor** (Supervisar) de la izquierda. Verá que la ejecución de la canalización que desencadenó manualmente. Use el botón **Refresh** (Actualizar) para actualizar la lista.

    ![Ejecución de canalización incorrecta](./media/tutorial-control-flow-portal/monitor-failure-pipeline-run.png)
2. Haga clic en el vínculo **Error** para ver los detalles del error en la ejecución de la canalización.

    ![Error en la canalización](./media/tutorial-control-flow-portal/pipeline-error-message.png)
2. Para ver las **ejecuciones de actividad** asociadas con la de esta canalización, haga clic en el primer vínculo de la columna **Actions** (Acciones). Use el botón **Refresh** (Actualizar) para actualizar la lista. Tenga en cuenta que en la actividad de copia de la canalización se produjo un error. La actividad web envió correctamente el correo electrónico de operación incorrecta al receptor especificado.

    ![Ejecuciones de actividad](./media/tutorial-control-flow-portal/activity-runs-failure.png)
4. Haga clic en el vínculo **Error** de la columna **Actions** (Acciones) para ver los detalles sobre el error.

    ![Error de ejecución de la actividad](./media/tutorial-control-flow-portal/activity-run-error.png)

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, realizó los pasos siguientes:

> [!div class="checklist"]
> * Creación de una factoría de datos.
> * Creación de un servicio vinculado de Azure Storage
> * Creación de un conjunto de datos del blob de Azure
> * Creación de una canalización que contiene una actividad de copia y una actividad web
> * Envío de los resultados de las actividades en actividades subsiguientes
> * Uso de las variables del sistema y del paso de parámetros
> * Inicio de la ejecución de una canalización
> * Supervisión de las ejecuciones de canalización y actividad

Ahora puede continuar a la sección de conceptos para obtener más información sobre Azure Data Factory.
> [!div class="nextstepaction"]
>[Canalizaciones y actividades](concepts-pipelines-activities.md)
