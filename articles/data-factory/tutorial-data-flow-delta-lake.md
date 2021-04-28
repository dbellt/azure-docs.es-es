---
title: ETL de Delta Lake con flujos de datos
description: En este tutorial se proporcionan instrucciones paso a paso para el uso de flujos de datos para transformar y analizar datos en Delta Lake.
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2021
ms.date: 04/16/2021
ms.openlocfilehash: 4a88ed2df74d3eebb96c42e2cdc87b14153419cd
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107565379"
---
# <a name="transform-data-in-delta-lake-using-mapping-data-flows"></a>Transformación de datos en Delta Lake mediante flujos de datos de asignación

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Si no está familiarizado con Azure Data Factory, consulte [Introducción a Azure Data Factory](introduction.md).

En este tutorial, usará el lienzo de flujo de datos para crear flujos de datos que le permitan analizar y transformar datos en Azure Data Lake Storage (ADLS) Gen2 y almacenarlos en Delta Lake.

## <a name="prerequisites"></a>Prerrequisitos
* **Suscripción de Azure**. Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* **Cuenta de Azure Storage**. El almacenamiento ADLS se puede usar como almacén de datos de *origen* y *receptor*. Si no tiene una cuenta de almacenamiento, consulte [Crear una cuenta de almacenamiento](../storage/common/storage-account-create.md) para crear una.

El archivo que se está transformando en este tutorial es MoviesDB. csv, que se puede encontrar [aquí](https://github.com/kromerm/adfdataflowdocs/blob/master/sampledata/moviesDB2.csv). Para recuperar el archivo de GitHub, copie el contenido en un editor de texto de su elección para guardarlo localmente como un archivo. csv. Para cargar el archivo en la cuenta de almacenamiento, vea [Carga de blobs con Azure Portal](../storage/blobs/storage-quickstart-blobs-portal.md). Los ejemplos harán referencia a un contenedor denominado "Sample-Data".

## <a name="create-a-data-factory"></a>Crear una factoría de datos

En este paso, creará una factoría de datos y abrirá la interfaz de usuario de Data Factory para crear una canalización en la factoría de datos.

1. Abra **Microsoft Edge** o **Google Chrome**. Actualmente, la interfaz de usuario de Data Factory solo se admite en los exploradores web Microsoft Edge y Google Chrome.
1. En el menú de la izquierda, seleccione **Crear un recurso** > **Integración** > **Data Factory**.
1. En la página **Nueva factoría de datos**, en **Nombre**, escriba **ADFTutorialDataFactory**.
1. Seleccione la **suscripción** de Azure en la que quiere crear la factoría de datos.
1. Para **Grupo de recursos**, realice uno de los siguientes pasos:

    a. Seleccione en primer lugar **Usar existente** y después un grupo de recursos de la lista desplegable.

    b. Seleccione **Crear nuevo** y escriba el nombre de un grupo de recursos. 
         
    Para más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](../azure-resource-manager/management/overview.md). 
1. En **Versión**, seleccione **V2**.
1. En **Ubicación**, seleccione la ubicación de la factoría de datos. En la lista desplegable solo se muestran las ubicaciones que se admiten. Los almacenes de datos (por ejemplo, Azure Storage y SQL Database) y los procesos (por ejemplo, Azure HDInsight) que la factoría de datos usa pueden estar en otras regiones.
1. Seleccione **Crear**.
1. Una vez finalizada la creación, verá el aviso en el centro de notificaciones. Seleccione **Ir al recurso** para ir a la página de Data Factory.
1. Haga clic en **Author & Monitor** (Creación y supervisión) para iniciar la interfaz de usuario de Data Factory en una pestaña independiente.

## <a name="create-a-pipeline-with-a-data-flow-activity"></a>Creación de una canalización con una actividad de flujo de datos

En este paso, creará una canalización que contiene una actividad de flujo de datos.

1. En la página **Let's get started** (Introducción) seleccione **Create pipeline** (Crear canalización).

   ![Creación de una canalización](./media/doc-common-process/get-started-page.png)

1. En la pestaña **General** de la canalización, escriba **DeltaLake** en el campo **Nombre** de la canalización.
1. En el panel **Actividades** expanda el acordeón **Movimiento y transformación**. Arrastre y coloque la actividad **Data Flow** del panel al lienzo de la canalización.

    ![Captura de pantalla que muestra el lienzo de canalización donde puede colocar la actividad de Data Flow.](media/tutorial-data-flow/activity1.png)
1. En el menú emergente **Adding Data Flow** (Adición de Data Flow), seleccione **Create New Data Flow** (Crear nueva instancia de Data Flow) y, a continuación, asigne el nombre **DeltaLake** al flujo de datos. Haga clic en Finalizar cuando haya terminado.

    ![Captura de pantalla que muestra la ubicación donde se asigna nombre al flujo de datos al crear uno nuevo.](media/tutorial-data-flow/activity2.png)
1. En la barra superior del lienzo de la canalización, mueva el control deslizante **Depuración de flujo de datos** a la posición de activado. El modo de depuración permite realizar pruebas interactivas de la lógica de transformación en un clúster de Spark activo. Los clústeres de Data Flow tardan de 5 a 7 minutos en prepararse y se recomienda que los usuarios activen primero la depuración si planean realizar el desarrollo de Data Flow. Para más información, consulte [Modo de depuración](concepts-data-flow-debug-mode.md).

    ![Captura de pantalla que muestra el control deslizante Data flow debug (Depuración de flujo de datos).](media/tutorial-data-flow/dataflow1.png)

## <a name="build-transformation-logic-in-the-data-flow-canvas&quot;></a>Generación de la lógica de transformación en el lienzo de flujo de datos

En este tutorial se generarán dos flujos de datos. El primer flujo de datos es un origen simple de receptor para generar una nueva instancia de Delta Lake a partir del archivo CSV de películas anterior. Por último, creará el diseño de flujo siguiente para actualizar los datos en Delta Lake.

![Flujo final](media/data-flow/data-flow-tutorial-6.png &quot;Flujo final")

### <a name="tutorial-objectives"></a>Objetivos del tutorial

1. Tome el origen del conjunto de datos MoviesCSV anterior y forme una nueva instancia de Delta Lake a partir de este.
1. Compile la lógica para las clasificaciones actualizadas de películas de 1988 en "1".
1. Elimine todas las películas de 1950.
1. Inserte nuevas películas de 2021 mediante la duplicación de películas de 1960.

### <a name="start-from-a-blank-data-flow-canvas"></a>Comenzar con un lienzo de flujo de datos en blanco

1. Haga clic la transformación de origen.
1. Haga clic en Nuevo junto al conjunto de datos en el panel inferior 1 de creación de un nuevo servicio vinculado para ADLS Gen2.
1. Elija Texto delimitado como tipo de conjunto de datos.
1. Asigne el nombre "MoviesCSV" al conjunto de datos. 
1. Apunte al archivo MoviesCSV que cargó en el almacenamiento anterior.
1. Establézcalo en el formato delimitado por comas e incluya el encabezado en la primera fila. 
1. Vaya a la pestaña de proyección de origen y haga clic en "Detect data types" (Detectar tipos de datos).
1. Una vez que tenga la proyección establecida, puede continuar. 
1. Agregue una transformación de receptor.
1. Delta es un tipo de conjunto de datos en línea. Tendrá que apuntar a la cuenta de almacenamiento de ADLS Gen2.
   
   ![Conjunto de datos insertado](media/data-flow/data-flow-tutorial-5.png "Conjunto de datos insertado")

1. Elija un nombre de carpeta en el contenedor de almacenamiento donde desea que ADF cree la instancia de Delta Lake.
1. Vuelva al diseñador de canalizaciones y haga clic en Depurar para ejecutar la canalización en modo de depuración solo con esta actividad de flujo de datos en el lienzo. Se generará la nueva instancia de Delta Lake en ADLS Gen2.
1. Desde Recursos de fábrica, haga clic en Nuevo > Flujo de datos 
1. Vuelva a usar MoviesCSV como origen y haga clic de nuevo en "Detect data types" (Detectar tipos de datos).
1. Agregue una transformación de filtro a la transformación de origen en el gráfico.
1. Permita solo las filas de película que coincidan con los tres años con los que va a trabajar, que serán 1950, 1988 y 1960.
1. Actualice las clasificaciones de cada película de 1988 a "1" con la adición de una transformación de columna derivada a la transformación del filtro.
1. En esa misma columna derivada, cree películas para 2021. Para ello, tome un año existente y cambie el año a 2021. Vamos a elegir 1960.
1. Este es el aspecto que tendrán las tres columnas derivadas:

   ![Columna derivada](media/data-flow/data-flow-tutorial-2.png "Columna derivada")
   
1. Las directivas ```Update, insert, delete, and upsert``` se crean en la transformación de alteración de fila. Agregue una transformación de alteración de fila después de la columna derivada.
1. Las directivas de alteración de fila deben tener este aspecto.

   ![Alteración de fila](media/data-flow/data-flow-tutorial-3.png "Alteración de fila")
   
1. Ahora que ha establecido la directiva adecuada para cada tipo de alteración de fila, compruebe que se han establecido las reglas de actualización adecuadas en la transformación de receptor.

   ![Sink](media/data-flow/data-flow-tutorial-4.png "Receptor")
   
1. Aquí vamos a usar el receptor de Delta Lake en el lago de datos de ADLS Gen2 y permitiremos inserciones, actualizaciones y eliminaciones. 
1. Observe que las columnas de clave son una clave compuesta formada por la columna de clave principal "movie" y la columna "year". Esto se debe a que hemos creado películas de 2021 falsas mediante la duplicación de las filas de 1960. Esto evita las colisiones, ya que proporciona unicidad al consultar las filas existentes.

### <a name="download-completed-sample"></a>Descarga del ejemplo completado
[Esta es una solución de ejemplo para la canalización Delta con un flujo de datos para actualizar o eliminar filas en el lago:](https://github.com/kromerm/adfdataflowdocs/blob/master/sampledata/DeltaPipeline.zip)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre el [lenguaje de expresiones de flujo de datos](data-flow-expression-functions.md).
