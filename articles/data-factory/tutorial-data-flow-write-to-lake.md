---
title: Procedimientos recomendados para escribir en archivos en un lago de datos con flujos de datos
description: Este tutorial contiene los procedimientos recomendados para escribir en archivos en un lago de datos con flujos de datos.
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2021
ms.date: 04/01/2021
ms.openlocfilehash: 8010f3f95c9358714b659df5821a375bd8488ad8
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106582100"
---
# <a name="best-practices-for-writing-to-files-to-data-lake-with-data-flows"></a>Procedimientos recomendados para escribir en archivos en un lago de datos con flujos de datos

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Si no está familiarizado con Azure Data Factory, consulte [Introducción a Azure Data Factory](introduction.md).

En este tutorial, encontrará los procedimientos recomendados que se pueden aplicar al escribir archivos en ADLS Gen2 o Azure Blob Storage mediante flujos de datos. Necesitará acceso a una cuenta de Azure Blob Storage o de Azure Data Lake Store Gen2 para leer un archivo Parquet y, a continuación, almacenar los resultados en carpetas.

## <a name="prerequisites"></a>Requisitos previos
* **Suscripción de Azure**. Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* **Cuenta de Azure Storage**. El almacenamiento ADLS se puede usar como almacén de datos de *origen* y *receptor*. Si no tiene una cuenta de almacenamiento, consulte [Crear una cuenta de almacenamiento](../storage/common/storage-account-create.md) para crear una.

Los pasos de este tutorial dan por hecho que tiene: 

## <a name="create-a-data-factory"></a>Crear una factoría de datos

En este paso, creará una factoría de datos y abrirá la interfaz de usuario de Data Factory para crear una canalización en la factoría de datos.

1. Abra **Microsoft Edge** o **Google Chrome**. Actualmente, la interfaz de usuario de Data Factory solo se admite en los exploradores web Microsoft Edge y Google Chrome.
1. En el menú de la izquierda, seleccione **Crear un recurso** > **Integración** > **Data Factory**.
1. En la página **Nueva factoría de datos**, en **Nombre**, escriba **ADFTutorialDataFactory**.
1. Seleccione la **suscripción** de Azure en la que quiere crear la factoría de datos.
1. Para **Grupo de recursos**, realice uno de los siguientes pasos:

    a. Seleccione en primer lugar **Usar existente** y después un grupo de recursos de la lista desplegable.
    
    b. Seleccione **Crear nuevo** y especifique un nombre de grupo de recursos. Para obtener más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](../azure-resource-manager/management/overview.md).
    
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
1. En la barra superior de Data Factory, deslice el control deslizante **Depuración de Data Flow** para activarlo. El modo de depuración permite realizar pruebas interactivas de la lógica de transformación en un clúster de Spark activo. Los clústeres de Data Flow tardan de 5 a 7 minutos en prepararse y se recomienda que los usuarios activen primero la depuración si planean realizar el desarrollo de Data Flow. Para más información, consulte [Modo de depuración](concepts-data-flow-debug-mode.md).

    ![Actividad de Data Flow](media/tutorial-data-flow/dataflow1.png)
1. En el panel **Actividades** expanda el acordeón **Movimiento y transformación**. Arrastre y coloque la actividad **Data Flow** del panel al lienzo de la canalización.

    ![Captura de pantalla que muestra el lienzo de canalización donde puede colocar la actividad de Data Flow.](media/tutorial-data-flow/activity1.png)
1. En el menú emergente **Adding Data Flow** (Adición de Data Flow), seleccione **Create New Data Flow** (Crear nueva instancia de Data Flow) y, a continuación, asigne el nombre **DeltaLake** al flujo de datos. Haga clic en Finalizar cuando haya terminado.

    ![Captura de pantalla que muestra la ubicación donde se asigna nombre al flujo de datos al crear uno nuevo.](media/tutorial-data-flow/activity2.png)

## <a name="build-transformation-logic-in-the-data-flow-canvas"></a>Generación de la lógica de transformación en el lienzo de flujo de datos

Tomará cualquier dato de origen (en este tutorial, usaremos un origen de archivos Parquet) y usará una transformación de receptor para que los datos lleguen en formato Parquet mediante los mecanismos más eficaces para ETL de lago de datos.

![Flujo final](media/data-flow/parts-final.png "Flujo final")

### <a name="tutorial-objectives"></a>Objetivos del tutorial

1. Elija cualquiera de sus conjuntos de datos de origen en un nuevo flujo de datos.
1. Utilice flujos de datos para hacer una partición eficaz de su conjunto de datos de receptor.
1. Envíe los datos con particiones a carpetas del lago de ADLS Gen2.

### <a name="start-from-a-blank-data-flow-canvas"></a>Comenzar con un lienzo de flujo de datos en blanco

En primer lugar, vamos a configurar el entorno de flujo de datos para cada uno de los mecanismos que se describen a continuación para enviar datos a ADLS Gen2.

1. Haga clic en la transformación de origen.
1. Haga clic en el botón Nuevo situado junto al conjunto de datos en el panel inferior.
1. Elija un conjunto de datos o cree uno nuevo. Para esta demo, usaremos un conjunto de datos de Parquet denominado datos de usuario.
1. Agregue una transformación de columna derivada. Lo utilizaremos como una forma de establecer los nombres de carpeta que desee de una manera dinámica.
1. Agregue una transformación de receptor.
   
### <a name="hierarchical-folder-output"></a>Salida de carpeta jerárquica

Es muy común usar valores únicos en los datos para crear particiones de los datos en jerarquías de carpetas en el lago. Es una muy buena forma de organizar y procesar datos en el lago y en Spark (el motor de proceso que hay detrás de los flujos de datos). Sin embargo, organizar la salida de esta manera tendrá un pequeño costo de rendimiento. Observará una leve disminución en el rendimiento general de la canalización al usar este mecanismo en el receptor.

1. Vuelva al diseñador del flujo de datos y modifique el flujo de datos creado anteriormente. Haga clic en la transformación del receptor.
1. Haga clic en Optimizar > Establecer partición > Clave.
1. Escoja la columna o columnas para establecer la estructura jerárquica de las carpetas.
1. Tenga en cuenta que el ejemplo siguiente utiliza el año y el mes como columnas para la nomenclatura de las carpetas. Los resultados serán carpetas con el formato ```releaseyear=1990/month=8```.
1. Al acceder a las particiones de datos en un origen de flujo de datos, apuntará solo a la carpeta de nivel superior anterior ```releaseyear``` y usará un patrón de caracteres comodín para cada carpeta posterior, por ejemplo: ```**/**/*.parquet```
1. Para modificar los valores de los datos, o incluso si es necesario generar valores sintéticos para los nombres de carpeta, use la transformación de columna derivada para crear los valores que desea usar en los nombres de carpeta.

![Creación de particiones de clave](media/data-flow/key-parts.png "Creación de particiones de clave")
   
### <a name="name-folder-as-data-values"></a>Nombre de carpeta como valores de datos

Hay una técnica de receptor que tiene un rendimiento ligeramente mejor para los datos de lago con ADLS Gen2, pero que no ofrece las mismas ventajas que la creación de particiones de clave-valor, y es ```Name folder as column data```. Mientras que el estilo de creación de particiones de clave de la estructura jerárquica le permitirá procesar segmentos de datos más fácilmente, esta técnica es una estructura de carpetas aplanada que puede escribir datos con más rapidez.

1. Vuelva al diseñador del flujo de datos y modifique el flujo de datos creado anteriormente. Haga clic en la transformación del receptor.
1. Haga clic en Optimizar > Establecer creación de particiones > Utilizar la creación de particiones actual.
1. Haga clic en Configuración > Asignar nombre a la carpeta como datos de columna.
1. Elija la columna que desea utilizar para generar nombres de carpeta.
1. Para modificar los valores de los datos, o incluso si es necesario generar valores sintéticos para los nombres de carpeta, use la transformación de columna derivada para crear los valores que desea usar en los nombres de carpeta.

![Opción carpeta](media/data-flow/folders.png "Carpetas")

### <a name="name-file-as-data-values"></a>Nombre de archivo como valores de datos

Las técnicas mencionadas en los tutoriales anteriores son casos de uso adecuados para crear categorías de carpetas en el lago de datos. El esquema de nomenclatura de archivos predeterminado que emplean esas técnicas es usar el identificador de trabajo del ejecutor de Spark. Algunas veces querrá establecer el nombre del archivo de salida en un receptor de texto de flujo de datos. Se recomienda el uso de esta técnica solo para archivos pequeños. El proceso de combinar archivos de partición en un único archivo de salida es un proceso de ejecución prolongada.

1. Vuelva al diseñador del flujo de datos y modifique el flujo de datos creado anteriormente. Haga clic en la transformación del receptor.
1. Haga clic en Optimizar > Establecer creación de particiones > Partición única. Este requisito de partición única crea un cuello de botella en el proceso de ejecución a medida que se combinan los archivos. Esta opción solo se recomienda para archivos pequeños.
1. Haga clic en Configuración > Asignar nombre al archivo como datos de columna.
1. Elija la columna que desea utilizar para generar nombres de archivo.
1. Para modificar los valores de los datos, o incluso si es necesario generar valores sintéticos para los nombres de archivo, use la transformación de columna derivada para crear los valores que desea usar en los nombres de archivo.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre los [receptores de flujo de datos](data-flow-sink.md).
