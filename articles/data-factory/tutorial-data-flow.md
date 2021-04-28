---
title: Transformación de datos mediante un flujo de datos de asignación
description: Este tutorial proporciona instrucciones detalladas para usar Azure Data Factory con el objetivo de transformar los datos con un flujo de datos de asignación.
author: dcstwh
ms.author: weetok
ms.reviewer: makromer
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 04/16/2021
ms.openlocfilehash: f8570c8b252fae91986508abd3725cbbedd361a0
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107565430"
---
# <a name="transform-data-using-mapping-data-flows"></a>Transformación de datos mediante flujos de datos de asignación

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Si no está familiarizado con Azure Data Factory, consulte [Introducción a Azure Data Factory](introduction.md).

En este tutorial, usará la interfaz de usuario de Azure Data Factory (UX) para crear una canalización que copie y transforme los datos desde un origen de Azure Data Lake Storage (ADLS) Gen2 hasta un receptor de ADLS Gen2 mediante un flujo de datos de asignación. El patrón de configuración de este tutorial se puede expandir al transformar los datos mediante el flujo de datos de asignación.

 >[!NOTE]
   >Este tutorial está pensado para los flujo de datos de asignación en general. Los flujos de datos están disponibles en las canalizaciones Azure Data Factory y Synapse. Si no está familiarizado con los flujos de datos en las canalizaciones de Azure Synapse, siga el [flujo de datos con las canalizaciones de Azure Synapse](../synapse-analytics/concepts-data-flow-overview.md). 
   
En este tutorial, realizará los siguientes pasos:

> [!div class="checklist"]
> * Creación de una factoría de datos.
> * Creación de una canalización con una actividad de Data Flow.
> * Cree un flujo de datos de asignación con cuatro transformaciones.
> * Realización de la serie de pruebas de la canalización.
> * Supervisión de una actividad de Data Flow

## <a name="prerequisites"></a>Requisitos previos
* **Suscripción de Azure**. Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* **Cuenta de Azure Storage**. El almacenamiento ADLS se puede usar como almacén de datos de *origen* y *receptor*. Si no tiene una cuenta de almacenamiento, consulte [Crear una cuenta de almacenamiento](../storage/common/storage-account-create.md) para crear una.

El archivo que se está transformando en este tutorial es MoviesDB. csv, que se puede encontrar [aquí](https://raw.githubusercontent.com/djpmsft/adf-ready-demo/master/moviesDB.csv). Para recuperar el archivo de GitHub, copie el contenido en un editor de texto de su elección para guardarlo localmente como un archivo. csv. Para cargar el archivo en la cuenta de almacenamiento, vea [Carga de blobs con Azure Portal](../storage/blobs/storage-quickstart-blobs-portal.md). Los ejemplos harán referencia a un contenedor denominado "Sample-Data".

## <a name="create-a-data-factory"></a>Crear una factoría de datos

En este paso, creará una factoría de datos y abrirá la interfaz de usuario de Data Factory para crear una canalización en la factoría de datos.

1. Abra **Microsoft Edge** o **Google Chrome**. Actualmente, la interfaz de usuario de Data Factory solo se admite en los exploradores web Microsoft Edge y Google Chrome.
2. En el menú de la izquierda, seleccione **Crear un recurso** > **Integración** > **Data Factory**:

   ![Selección de la factoría de datos en el panel Nuevo](./media/doc-common-process/new-azure-data-factory-menu.png)

3. En la página **Nueva factoría de datos**, en **Nombre**, escriba **ADFTutorialDataFactory**.

   El nombre de la instancia de Azure Data Factory debe ser *único de forma global*. Si recibe un mensaje de error sobre el valor de nombre, escriba un nombre diferente para la factoría de datos. (Por ejemplo, utilice SuNombreADFTutorialDataFactory). Para conocer las reglas de nomenclatura de los artefactos de Data Factory, consulte [Azure Data Factory: reglas de nomenclatura](naming-rules.md).

    :::image type="content" source="./media/doc-common-process/name-not-available-error.png" alt-text="Nuevo mensaje de error de factoría de datos por nombre duplicado.":::
4. Seleccione la **suscripción** de Azure en la que quiere crear la factoría de datos.
5. Para **Grupo de recursos**, realice uno de los siguientes pasos:

    a. Seleccione en primer lugar **Usar existente** y después un grupo de recursos de la lista desplegable.

    b. Seleccione **Crear nuevo** y escriba el nombre de un grupo de recursos. 
         
    Para más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](../azure-resource-manager/management/overview.md). 
6. En **Versión**, seleccione **V2**.
7. En **Ubicación**, seleccione la ubicación de la factoría de datos. En la lista desplegable solo se muestran las ubicaciones que se admiten. Los almacenes de datos (por ejemplo, Azure Storage y SQL Database) y los procesos (por ejemplo, Azure HDInsight) que la factoría de datos usa pueden estar en otras regiones.
8. Seleccione **Crear**.
9. Una vez finalizada la creación, verá el aviso en el centro de notificaciones. Seleccione **Ir al recurso** para ir a la página de Data Factory.
10. Haga clic en **Author & Monitor** (Creación y supervisión) para iniciar la interfaz de usuario de Data Factory en una pestaña independiente.

## <a name="create-a-pipeline-with-a-data-flow-activity"></a>Creación de una canalización con una actividad de Data Flow

En este paso, creará una canalización que contiene una actividad de Data Flow.

1. En la página **Let's get started** (Introducción) seleccione **Create pipeline** (Crear canalización).

   ![Creación de una canalización](./media/doc-common-process/get-started-page.png)

1. En la pestaña **General** de la canalización, escriba **TransformMovies** como **Nombre** de la canalización.
1. En el panel **Actividades** expanda el acordeón **Movimiento y transformación**. Arrastre y coloque la actividad **Data Flow** del panel al lienzo de la canalización.

    ![Captura de pantalla que muestra el lienzo de canalización donde puede colocar la actividad de Data Flow.](media/tutorial-data-flow/activity1.png)
1. En el menú emergente **Adding Data Flow** (Agregando Data Flow), seleccione **Create New Data Flow** (Crear nuevo Data Flow) y, a continuación, asigne el nombre **TransformMovies** al flujo de datos. Haga clic en Finalizar cuando haya terminado.

    ![Captura de pantalla que muestra la ubicación donde se asigna nombre al flujo de datos al crear uno nuevo.](media/tutorial-data-flow/activity2.png)
1. En la barra superior del lienzo de la canalización, mueva el control deslizante **Depuración de flujo de datos** a la posición de activado. El modo de depuración permite realizar pruebas interactivas de la lógica de transformación en un clúster de Spark activo. Los clústeres de Data Flow tardan de 5 a 7 minutos en prepararse y se recomienda que los usuarios activen primero la depuración si planean realizar el desarrollo de Data Flow. Para más información, consulte [Modo de depuración](concepts-data-flow-debug-mode.md).

    ![Actividad de Data Flow](media/tutorial-data-flow/dataflow1.png)

## <a name="build-transformation-logic-in-the-data-flow-canvas"></a>Generación de la lógica de transformación en el lienzo de flujo de datos

Una vez creado el flujo de datos, se le enviará automáticamente al lienzo flujo de datos. En este paso, creará un flujo de datos que toma el archivo moviesDB. csv en el almacenamiento de ADLS y agrega el promedio de clasificación de Comedias de 1910 a 2000. Después, volverá a escribir este archivo en el almacenamiento de ADLS.

1. Para agregar un lienzo de flujo de datos, haga clic en el cuadro **Agregar origen** y agregue un origen.

    ![Captura de pantalla que muestra el cuadro Agregar origen.](media/tutorial-data-flow/dataflow2.png)
1. Asigne un nombre al origen **MoviesDB**. Haga clic en **Nuevo** para crear un conjunto de datos de origen nuevo.

    ![Captura de pantalla que muestra la ubicación donde se selecciona Nuevo después de asignar un nombre al origen.](media/tutorial-data-flow/dataflow3.png)
1. Elija **Azure Data Lake Storage Gen2**. Haga clic en Continue.

    ![Captura de pantalla que muestra el mosaico de Azure Data Lake Storage Gen2.](media/tutorial-data-flow/dataset1.png)
1. Elija **DelimitedText**. Haga clic en Continue.

    ![Captura de pantalla que muestra el icono de DelimitedText.](media/tutorial-data-flow/dataset2.png)
1. Asigne un nombre al conjunto de datos **MoviesDB**. En la lista desplegable de servicios vinculados, elija **Nuevo**.

    ![Captura de pantalla que muestra la lista desplegable Servicio vinculado.](media/tutorial-data-flow/dataset3.png)
1. En la pantalla de creación de un servicio vinculado, asigne el nombre **ADLSGen2** al servicio vinculado ADLS gen2 y especifique el método de autenticación. A continuación, escriba las credenciales de conexión. En este tutorial, vamos a usar la clave de cuenta para conectarse a la cuenta de almacenamiento. Puede hacer clic en **Prueba de conexión** para comprobar que las credenciales se escribieron correctamente. Cuando haya terminado, haga clic en Crear.

    ![Servicios vinculados](media/tutorial-data-flow/ls1.png)
1. Una vez que vuelva a la pantalla de creación del conjunto de datos, escriba la ubicación del archivo en el campo **Ruta de acceso de archivo**. En este tutorial, el archivo moviesDB.csv se encuentra en el contenedor sample-data. Como el archivo tiene encabezados, active **First row as header** (Primera fila como encabezado). Seleccione **From Connection/Store** (Desde la conexión o almacén) para importar el esquema de encabezado directamente desde el archivo en el almacenamiento. Haga clic en Aceptar cuando haya terminado.

    ![Conjuntos de datos](media/tutorial-data-flow/dataset4.png)
1. Si se ha iniciado el clúster de depuración, vaya a la pestaña **Vista previa de los datos** de la transformación de origen y haga clic en **Actualizar** para obtener una instantánea de los datos. Puede usar la vista previa de los datos para comprobar que la transformación está configurada correctamente.

    ![Captura de pantalla que muestra la ubicación donde se puede obtener una vista previa de los datos para comprobar que la transformación está configurada correctamente.](media/tutorial-data-flow/dataflow4.png)
1. Junto al nodo de origen en el lienzo de flujo de datos, haga clic en el icono de signo más para agregar una nueva transformación. La primera transformación que va a agregar es un **filtro**.

    ![Lienzo de Data Flow](media/tutorial-data-flow/dataflow5.png)
1. Denomine **FilterYears** a la transformación de filtro. Haga clic en el cuadro de expresión junto a **Filtro en** para abrir el generador de expresiones. Aquí especificará la condición de filtrado.

    ![Captura de pantalla que muestra el cuadro de expresión Filtro en.](media/tutorial-data-flow/filter1.png)
1. El generador de expresiones de flujo de datos le permite compilar de forma interactiva expresiones para utilizarlas en varias transformaciones. Las expresiones pueden incluir funciones integradas, columnas del esquema de entrada y parámetros definidos por el usuario. Para más información sobre cómo compilar expresiones, vea [Generador de expresiones de Mapping Data Flow](concepts-data-flow-expression-builder.md).

    En este tutorial, desea filtrar las películas del género de comedia que se estrenaron entre los años 1910 y 2000. Dado que el año es actualmente una cadena, debe convertirlo en un entero mediante la función ```toInteger()```. Use los operadores mayor o igual que (> =) y menor o igual que (< =) para comparar con los valores de año literal 1910 y 200-. Una estas expresiones junto con el operador and (&&). La expresión aparece como:

    ```toInteger(year) >= 1910 && toInteger(year) <= 2000```

    Para averiguar qué películas son comedias, puede usar la función ```rlike()``` para buscar el patrón " comedia" en la columna de géneros. Una la expresión rlike con la comparación de año para obtener:

    ```toInteger(year) >= 1910 && toInteger(year) <= 2000 && rlike(genres, 'Comedy')```

    Si tiene un clúster de depuración activo, puede comprobar la lógica; para ello, haga clic en **Actualizar** para ver la salida de la expresión en comparación con las entradas usadas. Hay más de una respuesta correcta sobre cómo puede realizar esta lógica mediante el lenguaje de expresiones de flujo de datos.

    ![Filter](media/tutorial-data-flow/filter2.png)

    Haga clic en **Guardar y finalizar** una vez que haya terminado con la expresión.

1. Capture una **Vista previa de datos** para comprobar que el filtro funciona correctamente.

    ![Captura de pantalla que muestra la vista previa de datos que ha capturado.](media/tutorial-data-flow/filter3.png)
1. La transformación siguiente que se va a agregar es una transformación de **agregado** en **Schema Modifier** (Modificador de esquema).

    ![Captura de pantalla que muestra el modificador de esquema de agregado.](media/tutorial-data-flow/agg1.png)
1. Denomine **AggregateComedyRatings** a la transformación de agregado. En la pestaña **Agrupar por**, seleccione **year** (año) en la lista desplegable para agrupar las agregaciones por el año en que apareció la película.

    ![Captura de pantalla que muestra la opción año en la pestaña Agrupar por de Configuración de agregación.](media/tutorial-data-flow/agg2.png)
1. Vaya a la pestaña **Agregados**. En el cuadro de texto de la izquierda, asigne a la columna agregada el nombre **AverageComedyRating**. Haga clic en el cuadro de la expresión derecha para especificar la expresión de agregado a través del generador de expresiones.

    ![Captura de pantalla que muestra la opción año en la pestaña Agregados de Configuración de agregación.](media/tutorial-data-flow/agg3.png)
1. Para obtener el promedio de la columna **Rating** (Clasificación), use la función de agregado ```avg()```. Como **Rating** (Clasificación) es una cadena y ```avg()``` toma una entrada numérica, debemos convertir el valor en un número a través de la función ```toInteger()```. Se trata de una expresión similar a la siguiente:

    ```avg(toInteger(Rating))```

    Haga clic en **Guardar y finalizar** cuando haya terminado.

    ![Captura de pantalla que muestra la expresión guardada.](media/tutorial-data-flow/agg4.png)
1. Vaya a la pestaña **Vista previa de datos** para ver la salida de la transformación. Observe que solo hay dos columnas, **year** y **AverageComedyRating**.

    ![Agregado](media/tutorial-data-flow/agg3.png)
1. A continuación, desea agregar una transformación de **receptor** en **Destino**.

    ![Captura de pantalla que muestra la ubicación donde se va a agregar una transformación de receptor en Destino.](media/tutorial-data-flow/sink1.png)
1. Asigne un nombre al receptor **Sink** (Receptor). Haga clic en **Nuevo** para crear un conjunto de datos de receptor.

    ![Captura de pantalla que muestra la ubicación donde se puede asignar nombre al receptor y crear un nuevo conjunto de datos de receptor.](media/tutorial-data-flow/sink2.png)
1. Elija **Azure Data Lake Storage Gen2**. Haga clic en Continue.

    ![Captura de pantalla que muestra el icono de Azure Data Lake Storage Gen2 que se puede elegir.](media/tutorial-data-flow/dataset1.png)
1. Elija **DelimitedText**. Haga clic en Continue.

    ![Dataset](media/tutorial-data-flow/dataset2.png)
1. Asigne el nombre **MoviesSink** al conjunto de datos de receptor. Para el servicio vinculado, elija el servicio vinculado ADLS gen2 que creó en el paso 6. Escriba una carpeta de salida en la que escribir los datos. En este tutorial, vamos a escribir en la carpeta "output" (salida) en el contenedor "sample-data" (datos de ejemplo). No es necesario que la carpeta exista de antemano y se puede crear dinámicamente. Establezca **Usar la primera fila como encabezado** en true y seleccione **Ninguna** en **Importar esquema**. Haga clic en Finish.

    ![Receptor](media/tutorial-data-flow/sink3.png)

Ahora ha terminado de crear el flujo de datos. Está preparado para ejecutarlo en la canalización.

## <a name="running-and-monitoring-the-data-flow"></a>Ejecución y supervisión del flujo de datos

Puede depurar una canalización antes de publicarla. En este paso, va a desencadenar una ejecución de depuración de la canalización de flujo de datos. Aunque la vista previa de los datos no escribe datos, una ejecución de depuración escribirá datos en el destino del receptor.

1. Vaya al lienzo de la canalización. Haga clic en **Depurar** para desencadenar una ejecución de depuración.

    ![Captura de pantalla que muestra el lienzo de canalización con Depurar resaltado.](media/tutorial-data-flow/pipeline1.png)
1. La depuración de canalización de actividades de Data Flow usa el clúster de depuración activo, pero sigue tardando al menos un minuto en inicializarse. Puede realizar un seguimiento del progreso a través de la pestaña **Salida**. Una vez que la ejecución se realice correctamente, haga clic en el icono de anteojos para abrir el panel de supervisión.

    ![Canalización](media/tutorial-data-flow/pipeline2.png)
1. En el panel supervisión, puede ver el número de filas y el tiempo invertido en cada paso de transformación.

    ![Captura de pantalla que muestra el panel de supervisión donde se puede ver el número de filas y el tiempo invertido en cada paso de transformación.](media/tutorial-data-flow/pipeline3.png)
1. Haga clic en una transformación para obtener información detallada sobre las columnas y las particiones de los datos.

    ![Supervisión](media/tutorial-data-flow/pipeline4.png)

Si siguió correctamente este tutorial, debe haber escrito 83 filas y 2 columnas en la carpeta del receptor. Puede comprobar el almacenamiento de blobs para confirmar que los datos son correctos.

## <a name="next-steps"></a>Pasos siguientes

La canalización de este tutorial ejecuta un flujo de datos que agrega el promedio de la clasificación de comedias de 1910 a 2000 y escribe los datos en ADLS. Ha aprendido a:

> [!div class="checklist"]
> * Creación de una factoría de datos.
> * Creación de una canalización con una actividad de Data Flow.
> * Cree un flujo de datos de asignación con cuatro transformaciones.
> * Realización de la serie de pruebas de la canalización.
> * Supervisión de una actividad de Data Flow

Obtenga más información sobre el [lenguaje de expresiones de flujo de datos](data-flow-expression-functions.md).