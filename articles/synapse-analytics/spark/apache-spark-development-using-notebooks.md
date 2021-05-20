---
title: Uso de cuadernos de Synapse
description: En este artículo aprenderá a crear y desarrollar cuadernos de Synapse para la preparación y visualización de datos.
services: synapse analytics
author: ruixinxu
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: spark
ms.date: 05/08/2021
ms.author: ruxu
ms.reviewer: ''
ms.custom: devx-track-python
ms.openlocfilehash: a66b036bde5f25873e9d4a371faf249deadd69dc
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109736905"
---
# <a name="create-develop-and-maintain-synapse-notebooks-in-azure-synapse-analytics"></a>Creación, desarrollo y mantenimiento de cuadernos de Synapse en Azure Synapse Analytics

Un cuaderno de Synapse es una interfaz web para crear archivos que contengan código activo, visualizaciones y texto narrativo. Los cuadernos son un buen lugar para validar ideas y aplicar experimentos rápidos para sacar conclusiones a partir de los datos. Los cuadernos también se usan ampliamente en la preparación de datos, la visualización de datos, el aprendizaje automático y otros escenarios de macrodatos.

Con un cuaderno de Synapse, puede hacer lo siguiente:

* Empezar a trabajar sin esfuerzo alguno de configuración.
* Mantener los datos protegidos con las características de seguridad empresarial integradas.
* Analizar datos en formatos sin procesar (CSV, TXT, JSON, etc.), formatos de archivos procesados (parquet, Delta Lake, ORC, etc.) y archivos de datos tabulares de SQL en Spark y SQL.
* Ser productivo con funcionalidades de creación mejoradas y visualización de datos integrada.

En este artículo se describe cómo usar cuadernos en Synapse Studio.

## <a name="preview-of-the-new-notebook-experience"></a>Versión preliminar de la nueva experiencia de cuaderno
El equipo de Synapse proporciona el nuevo componente de cuaderno en Synapse Studio para ofrecer una experiencia de cuaderno coherente a los clientes de Microsoft y maximizar la capacidad de detección, la productividad, el uso compartido y la colaboración entre usuarios. La versión preliminar de la nueva experiencia de cuaderno ya está lista. En la barra de herramientas del cuaderno, pulse el botón **Características de versión preliminar** para activar la experiencia. En la tabla siguiente se captura la comparación de características entre el cuaderno existente (denominado "cuaderno clásico") y la nueva versión preliminar.  

|Característica|Cuaderno clásico|Versión preliminar del cuaderno|
|--|--|--|
|%run| No compatible | &#9745;|
|%history| No compatible |&#9745;|
|%load| No compatible |&#9745;|
|%%html| No compatible |&#9745;|
|Arrastrar y colocar para desplazar una celda| No compatible |&#9745;|
|Esquema (tabla de contenido)| No compatible |&#9745;|
|Explorador de variables| No compatible |&#9745;|
|Formato de celdas de texto con botones de la barra de herramientas|&#9745;| No disponible |
|Deshacer la operación de la celda| &#9745;| No disponible |


## <a name="create-a-notebook"></a>Creación de un cuaderno

Hay dos formas de crear un cuaderno. Puede crear un nuevo cuaderno o importar uno existente en un área de trabajo de Synapse desde el **Explorador de objetos**. Los cuadernos de Synapse reconocen los archivos IPYNB estándar de Jupyter Notebook.

![creación de un cuaderno de notas de importación](./media/apache-spark-development-using-notebooks/synapse-create-import-notebook-2.png)

## <a name="develop-notebooks"></a>Desarrollo de cuadernos

Los cuadernos se componen de celdas, que son bloques individuales de código o texto que se pueden ejecutar de manera independiente o en grupo.

### <a name="add-a-cell"></a>Adición de una celda

Hay varias maneras de agregar una nueva celda a un cuaderno.

# <a name="classical-notebook"></a>[Cuaderno clásico](#tab/classical)

1. Expanda el botón superior izquierdo **+ Celda** y seleccione **Agregar celda de código** o **Agregar celda de texto**.

    ![add-cell-with-cell-button](./media/apache-spark-development-using-notebooks/synapse-add-cell-1.png)

2. Mantenga el puntero sobre el espacio entre dos celdas y seleccione **Agregar código** o **Agregar texto**.

    ![add-cell-between-space](./media/apache-spark-development-using-notebooks/synapse-add-cell-2.png)

3. Utilice las [teclas de método abreviado en el modo de comando](#shortcut-keys-under-command-mode). Presione **A** para insertar una celda sobre la celda actual. Presione **B** para insertar una celda debajo de la celda actual.


# <a name="preview-notebook"></a>[Versión preliminar del cuaderno](#tab/preview)

1. Expanda el botón superior izquierdo **+ Celdas** y seleccione **celda de código** o **celda de Markdown**.
    ![add-azure-notebook-cell-with-cell-button](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-add-cell-1.png)
2. Seleccione el signo más (+) al principio de una celda y seleccione **celda de código** o **celda de Markdown**.

    ![add-azure-notebook-cell-between-space](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-add-cell-2.png)

3. Use las teclas de método abreviado [aznb en el modo de comando](#shortcut-keys-under-command-mode). Presione **A** para insertar una celda sobre la celda actual. Presione **B** para insertar una celda debajo de la celda actual.

---

### <a name="set-a-primary-language"></a>Definición del lenguaje principal

Los cuadernos de Synapse admiten cuatro lenguajes de Apache Spark:

* pySpark (Python)
* Spark (Scala)
* SparkSQL
* .NET para Apache Spark (C#)

Desde la lista desplegable de la barra de comandos superior, puede establecer el lenguaje principal para las nuevas celdas que se agreguen.

   ![default-synapse-language](./media/apache-spark-development-using-notebooks/synapse-default-language.png)

### <a name="use-multiple-languages"></a>Uso de varios lenguajes

Para usar varios lenguajes en un cuaderno, puede especificar el comando magic de lenguaje correcto al principio de una celda. En la tabla siguiente se enumeran los comandos magic para cambiar el lenguaje de las celdas.

|Comando magic |Idioma | Descripción |  
|---|------|-----|
|%%pyspark| Python | Ejecute una consulta de **Python** en SparkContext.  |
|%%spark| Scala | Ejecute una consulta de **Scala** en SparkContext.  |  
|%%sql| SparkSQL | Ejecute una consulta de **SparkSQL** en SparkContext.  |
|%%csharp | .NET para Spark C# | Ejecute una consulta de **.NET para Spark C#** en Spark Context. |

La imagen siguiente es un ejemplo de cómo se puede escribir una consulta de PySpark con el comando magic **%%pyspark** o una consulta de SparkSQL con el comando magic **%%sql** en un cuaderno de **Spark (Scala)** . Tenga en cuenta que el lenguaje principal del cuaderno está establecido en pySpark.

   ![Comandos magic de Spark de Synapse](./media/apache-spark-development-using-notebooks/synapse-spark-magics.png)

### <a name="use-temp-tables-to-reference-data-across-languages&quot;></a>Uso de tablas temporales para hacer referencia a datos entre lenguajes

No puede hacer referencia a datos o variables directamente entre distintos lenguajes en un cuaderno de Synapse. En Spark, se puede hacer referencia a una tabla temporal entre lenguajes. Este es un ejemplo de cómo leer un DataFrame de `Scala` en `PySpark` y `SparkSQL` mediante una tabla temporal de Spark como solución alternativa.

1. En la celda 1, lea un DataFrame de un conector de grupo de SQL mediante Scala y cree una tabla temporal.

   ```scala
   %%scala
   val scalaDataFrame = spark.read.sqlanalytics(&quot;mySQLPoolDatabase.dbo.mySQLPoolTable")
   scalaDataFrame.createOrReplaceTempView( "mydataframetable" )
   ```

2. En la celda 2, consulte los datos mediante Spark SQL.
   
   ```sql
   %%sql
   SELECT * FROM mydataframetable
   ```

3. En la celda 3, use los datos de PySpark.

   ```python
   %%pyspark
   myNewPythonDataFrame = spark.sql("SELECT * FROM mydataframetable")
   ```

### <a name="ide-style-intellisense"></a>IntelliSense de estilo IDE

Los cuadernos de Synapse se integran en el editor Monaco para incluir la funcionalidad IntelliSense de estilo IDE en el editor de celdas. El resaltado de la sintaxis, el marcador de errores y la finalización automática de código le ayudan a escribir código y a identificar problemas más rápido.

Las características de IntelliSense tienen distintos niveles de madurez para distintos lenguajes. Use la siguiente tabla para ver lo que se admite.

|Lenguajes| Resaltado de sintaxis | Creador de errores de sintaxis  | Finalización de código de sintaxis | Finalización de código de sintaxis| Finalización de código de funciones del sistema| Finalización de código de funciones del usuario| Sangría inteligente | Plegado de código|
|--|--|--|--|--|--|--|--|--|
|PySpark (Python)|Sí|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Spark (Scala)|Sí|Sí|Sí|Sí|-|-|-|Sí|
|SparkSQL|Sí|Sí|-|-|-|-|-|-|
|.NET para Spark ( C# )|Sí|-|-|-|-|-|-|-|



### <a name="code-snippets"></a>Fragmentos de código

Los cuadernos de Synapse proporcionan fragmentos de código que facilitan la introducción de patrones de código utilizados habitualmente, como la configuración de la sesión de Spark, la lectura de datos como dataframes de Spark o la creación de gráficos con matplotlib.

Los fragmentos de código aparecen en [IntelliSense](#ide-style-intellisense) combinados con otras sugerencias. El contenido de los fragmentos de código se alinea con el lenguaje de las celdas de código. Para ver los fragmentos de código disponibles, escriba **snippet** o cualquier palabra clave que aparezca en el título del fragmento de código en el editor de celdas de código. Por ejemplo, si escribe **read**, puede ver la lista de fragmentos de código para leer datos de varios orígenes de datos.

![Fragmentos de código de Synapse](./media/apache-spark-development-using-notebooks/synapse-code-snippets.gif#lightbox)



### <a name="format-text-cell-with-toolbar-buttons"></a>Formato de celdas de texto con botones de la barra de herramientas

# <a name="classical-notebook"></a>[Cuaderno clásico](#tab/classical)

Puede usar los botones de formato en la barra de herramientas de celdas de texto para completar acciones comunes de marcado. Incluye texto en negrita, cursiva, inserción de fragmentos de código, inserción de listas sin ordenar, inserción de listas ordenadas e inserción de imágenes desde direcciones URL.

  ![Barra de herramientas de celdas de texto de Synapse](./media/apache-spark-development-using-notebooks/synapse-text-cell-toolbar.png)

# <a name="preview-notebook"></a>[Versión preliminar del cuaderno](#tab/preview)

La barra de herramientas del botón de formato no está disponible aún para la versión preliminar de la experiencia de cuaderno. 

---

### <a name="undo-cell-operations"></a>Deshacer operaciones en celdas

# <a name="classical-notebook"></a>[Cuaderno clásico](#tab/classical)

Seleccione el botón **Deshacer** o presione **CTRL+Z** para revocar la operación más reciente en la celda. Ahora puede deshacer hasta las 20 acciones de celda históricas más recientes. 

   ![Deshacer celdas en Synapse](./media/apache-spark-development-using-notebooks/synapse-undo-cells.png)
# <a name="preview-notebook"></a>[Versión preliminar del cuaderno](#tab/preview)

La operación para deshacer una celda no está disponible aún en la versión preliminar de la experiencia de cuaderno. 

---

### <a name="move-a-cell"></a>Movimiento de una celda

# <a name="classical-notebook"></a>[Cuaderno clásico](#tab/classical)

Seleccione los puntos suspensivos (…) para acceder al menú de otras acciones de celda del extremo derecho. A continuación, seleccione **Move cell up** (Subir celda) o **Move cell down** (Bajar celda) para desplazar la celda actual. 

También puede utilizar las [teclas de método abreviado en el modo de comando](#shortcut-keys-under-command-mode). Presione **CTRL+Alt+↑** para subir la celda actual. Presione **CTRL+Alt+↓** para bajar la celda actual.

   ![move-a-cell](./media/apache-spark-development-using-notebooks/synapse-move-cells.png)

# <a name="preview-notebook"></a>[Versión preliminar del cuaderno](#tab/preview)

Haga clic en la parte izquierda de una celda y arrástrela hasta la posición deseada. 
    ![mover celdas en Synapse](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-drag-drop-cell.gif)

---

### <a name="delete-a-cell"></a>Eliminación de una celda

# <a name="classical-notebook"></a>[Cuaderno clásico](#tab/classical)

Para eliminar una celda, seleccione los puntos suspensivos (…) para acceder al menú de otras acciones de celda del extremo derecho y, luego, elija **Eliminar celda**. 

También puede utilizar las [teclas de método abreviado en el modo de comando](#shortcut-keys-under-command-mode). Presione **D,D** para eliminar la celda actual.
  
   ![delete-a-cell](./media/apache-spark-development-using-notebooks/synapse-delete-cell.png)

# <a name="preview-notebook"></a>[Versión preliminar del cuaderno](#tab/preview)

Para eliminar una celda, seleccione el botón Eliminar en la parte derecha de la celda. 

También puede utilizar las [teclas de método abreviado en el modo de comando](#shortcut-keys-under-command-mode). Presione **Mayús+D** para eliminar la celda actual. 

   ![azure-notebook-delete-a-cell](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-delete-cell.png)

---

### <a name="collapse-a-cell-input"></a>Contracción de una entrada de celda

# <a name="classical-notebook"></a>[Cuaderno clásico](#tab/classical)

Seleccione el botón de flecha situado en la parte inferior de la celda actual para contraerla. Para expandirla, seleccione el botón de flecha mientras la celda está contraída.

   ![collapse-cell-input](./media/apache-spark-development-using-notebooks/synapse-collapse-cell-input.gif)

# <a name="preview-notebook"></a>[Versión preliminar del cuaderno](#tab/preview)

Seleccione los puntos suspensivos (...) para ver **más comandos** en la barra de herramientas de la celda y la **entrada** para contraer la entrada de la celda actual. Para expandirla, seleccione la **entrada oculta** mientras la celda está contraída.

   ![azure-notebook-collapse-cell-input](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-collapse-cell-input.gif)

---

### <a name="collapse-a-cell-output"></a>Contracción de una salida de celda

# <a name="classical-notebook"></a>[Cuaderno clásico](#tab/classical)

Seleccione el botón **Contraer salida** situado en la parte superior izquierda de la salida de la celda actual para contraerla. Para expandirla, haga clic en **Mostrar salida de la celda** mientras la salida de la celda está contraída.

   ![collapse-cell-output](./media/apache-spark-development-using-notebooks/synapse-collapse-cell-output.gif)

# <a name="preview-notebook"></a>[Versión preliminar del cuaderno](#tab/preview)

Seleccione los puntos suspensivos (...) para ver **más comandos** en la barra de herramientas de la celda y la **salida** para contraer la salida de la celda actual. Para expandirla, seleccione el mismo botón mientras la salida de la celda está oculta.

   ![azure-notebook-collapse-cell-output](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-collapse-cell-output.gif)


---

### <a name="notebook-outline"></a>Esquema del cuaderno

# <a name="classical-notebook"></a>[Cuaderno clásico](#tab/classical)

No compatible.

# <a name="preview-notebook"></a>[Versión preliminar del cuaderno](#tab/preview)

En Esquema (tabla de contenido) se presenta el primer encabezado de cualquier celda de Markdown en una ventana de la barra lateral para permitir la navegación rápida. La barra lateral Esquemas es redimensionable y contraíble para ajustarse a la pantalla de la mejor manera posible. Puede seleccionar el botón **Esquema** de la barra de comandos del cuaderno para abrir u ocultar la barra lateral.

![azure-notebook-outline](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-outline.png)

---


## <a name="run-notebooks"></a>Ejecución de cuadernos

Puede ejecutar las celdas de código en el cuaderno individualmente o todas a la vez. El estado y el progreso de cada celda se representan en el cuaderno.

### <a name="run-a-cell"></a>Ejecución de una celda

Hay varias maneras de ejecutar el código de una celda.

1. Mantenga el puntero sobre la celda que desea ejecutar y seleccione el botón **Ejecutar celda** o presione **CTRL+Entrar**.

   ![run-cell-1](./media/apache-spark-development-using-notebooks/synapse-run-cell.png)
  
2. Utilice las [teclas de método abreviado en el modo de comando](#shortcut-keys-under-command-mode). Presione **Mayús+Entrar** para ejecutar la celda actual y seleccionar la celda a continuación. Presione **Mayús+Entrar** para ejecutar la celda actual e insertar una nueva celda a continuación.

---

### <a name="run-all-cells"></a>Ejecución de todas las celdas
Seleccione el botón **Ejecutar todo** para ejecutar todas las celdas del cuaderno actual en secuencia.

   ![run-all-cells](./media/apache-spark-development-using-notebooks/synapse-run-all.png)


### <a name="run-all-cells-above-or-below"></a>Ejecución de todas las celdas encima o debajo

# <a name="classical-notebook"></a>[Cuaderno clásico](#tab/classical)

Para acceder al menú de otras acciones de celda del extremo derecho, seleccione los puntos suspensivos ( **…** ) y, luego, elija **Run cells above** (Ejecutar las celdas encima) para ejecutar todas las celdas situadas encima de la actual en secuencia. Seleccione **Run cells below** (Ejecutar las celdas debajo) para ejecutar todas las celdas situadas debajo de la actual en secuencia.

   ![run-cells-above-or-below](./media/apache-spark-development-using-notebooks/synapse-run-cells-above-or-below.png)

# <a name="preview-notebook"></a>[Versión preliminar del cuaderno](#tab/preview)

Expanda la lista desplegable en el botón **Ejecutar todo** y, a continuación, seleccione **Run cells above** (Ejecutar las celdas encima) para ejecutar todas las celdas situadas encima de la actual en secuencia. Seleccione **Run cells below** (Ejecutar las celdas debajo) para ejecutar todas las celdas situadas debajo de la actual en secuencia.

   ![azure-notebook-run-cells-above-or-below](./media/apache-spark-development-using-notebooks/synapse-aznb-run-cells-above-or-below.png)

---

### <a name="cancel-all-running-cells"></a>Cancelación de todas las celdas en ejecución

# <a name="classical-notebook"></a>[Cuaderno clásico](#tab/classical)
Seleccione el botón **Cancelar todo** para cancelar las celdas en ejecución o las celdas que esperan en la cola. 
   ![cancel-all-cells](./media/apache-spark-development-using-notebooks/synapse-cancel-all.png) 

# <a name="preview-notebook"></a>[Versión preliminar del cuaderno](#tab/preview)

Seleccione el botón **Cancelar todo** para cancelar las celdas en ejecución o las celdas que esperan en la cola. 
   ![azure-notebook-cancel-all-cells](./media/apache-spark-development-using-notebooks/synapse-aznb-cancel-all.png) 

---



### <a name="notebook-reference"></a>Referencia del cuaderno

# <a name="classical-notebook"></a>[Cuaderno clásico](#tab/classical)

No compatible.

# <a name="preview-notebook"></a>[Versión preliminar del cuaderno](#tab/preview)

Puede usar el comando magic ```%run <notebook path>``` para hacer referencia a otro cuaderno en el contexto del cuaderno actual. Todas las variables definidas en el cuaderno de referencia están disponibles en el cuaderno actual. El comando magic ```%run``` admite llamadas anidadas pero no admite llamadas recursivas. Recibirá una excepción si la profundidad de la instrucción es superior a cinco. Actualmente, el comando ```%run``` solo se admite para pasar una ruta de acceso del cuaderno como parámetro. 

Ejemplo: ``` %run /path/notebookA ```.

> [!NOTE]
> La referencia del cuaderno no se admite en una canalización de Synapse.
>
>

---

### <a name="variable-explorer"></a>Explorador de variables

# <a name="classical-notebook"></a>[Cuaderno clásico](#tab/classical)

No compatible.

# <a name="preview-notebook"></a>[Versión preliminar del cuaderno](#tab/preview)

El cuaderno de Synapse ahora incluye un explorador de variables integrado para que pueda ver la lista de nombres, tipos, longitudes y valores de las variables en la sesión de Spark actual para las celdas de PySpark (Python). Se mostrarán automáticamente más variables a medida que se definan en las celdas de código. Al hacer clic en cada encabezado de columna se ordenarán las variables de la tabla.

Puede seleccionar el botón **Variables** de la barra de comandos del cuaderno para abrir u ocultar el explorador de variables.

![azure-notebook-variable-explorer](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-variable-explorer.png)


---

### <a name="cell-status-indicator"></a>Indicador de estado de la celda

Debajo de la celda se muestra un estado de ejecución detallado de la celda para ayudarle a ver su progreso actual. Una vez que se complete la ejecución de la celda, aparecerá un resumen de la ejecución con la duración total y la hora de finalización; este resumen se mantendrá allí para futuras referencias.

![cell-status](./media/apache-spark-development-using-notebooks/synapse-cell-status.png)

### <a name="spark-progress-indicator"></a>Indicador de progreso de Spark

Un cuaderno de Synapse se basa únicamente en Spark. Las celdas de código se ejecutan en el grupo de Apache Spark sin servidor de manera remota. Aparece un indicador de progreso del trabajo de Spark con una barra de progreso en tiempo real que le ayudará a entender el estado de ejecución del trabajo.
El número de tareas por cada trabajo o etapa ayuda a identificar el nivel paralelo del trabajo de Spark. También puede profundizar en la interfaz de usuario de Spark de un trabajo o fase específicos a través de la selección del vínculo del nombre del trabajo o de la fase.


![spark-progress-indicator](./media/apache-spark-development-using-notebooks/synapse-spark-progress-indicator.png)

### <a name="spark-session-config"></a>Configuración de la sesión de Spark

Puede especificar la duración del tiempo de espera, el número y el tamaño de los ejecutores que se van a dar a la sesión actual de Spark en **Configure session** (Configurar sesión). Reinicie la sesión de Spark para que surtan efecto los cambios de configuración. Se borran todas las variables del cuaderno almacenadas en la memoria caché.

[![session-management](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-spark-session-management.png)](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-spark-session-management.png#lightbox)

#### <a name="spark-session-config-magic-command"></a>Comando magic de configuración de la sesión de Spark
También puede especificar la configuración de la sesión de Spark a través de un comando magic **%%configure**. La sesión de Spark debe reiniciarse para que la configuración surta efecto. Se recomienda ejecutar el comando **%%configure** al principio del cuaderno. Recuerde que este es un ejemplo; para obtener una lista completa de parámetros válidos puede consultar https://github.com/cloudera/livy#request-body. 

```
%%configure -f
{
    to config the session.
    "driverMemory":"2g",
    "driverCores":3,
    "executorMemory":"2g",
    "executorCores":2,
    "jars":["myjar1.jar","myjar.jar"],
    "conf":{
        "spark.driver.maxResultSize":"10g"
    }
}
```
> [!NOTE]
> El comando magic de configuración de la sesión de Spark no se admite en una canalización de Synapse.
>
>

## <a name="bring-data-to-a-notebook"></a>Traslado de los datos a un cuaderno

Puede cargar datos desde Azure Blob Storage, Azure Data Lake Store Gen2 y el grupo de SQL, tal como se muestra en los ejemplos de código siguientes.

### <a name="read-a-csv-from-azure-data-lake-store-gen2-as-a-spark-dataframe"></a>Lectura de un archivo CSV desde Azure Data Lake Store Gen2 como DataFrame de Spark

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import *
account_name = "Your account name"
container_name = "Your container name"
relative_path = "Your path"
adls_path = 'abfss://%s@%s.dfs.core.windows.net/%s' % (container_name, account_name, relative_path)

df1 = spark.read.option('header', 'true') \
                .option('delimiter', ',') \
                .csv(adls_path + '/Testfile.csv')

```

#### <a name="read-a-csv-from-azure-blob-storage-as-a-spark-dataframe"></a>Lectura de un archivo CSV desde Azure Blob Storage como DataFrame de Spark

```python

from pyspark.sql import SparkSession

# Azure storage access info
blob_account_name = 'Your account name' # replace with your blob name
blob_container_name = 'Your container name' # replace with your container name
blob_relative_path = 'Your path' # replace with your relative folder path
linked_service_name = 'Your linked service name' # replace with your linked service name

blob_sas_token = mssparkutils.credentials.getConnectionStringOrCreds(linked_service_name)

# Allow SPARK to access from Blob remotely

wasb_path = 'wasbs://%s@%s.blob.core.windows.net/%s' % (blob_container_name, blob_account_name, blob_relative_path)

spark.conf.set('fs.azure.sas.%s.%s.blob.core.windows.net' % (blob_container_name, blob_account_name), blob_sas_token)
print('Remote blob path: ' + wasb_path)

df = spark.read.option("header", "true") \
            .option("delimiter","|") \
            .schema(schema) \
            .csv(wasbs_path)
```

### <a name="read-data-from-the-primary-storage-account"></a>Lectura de datos desde la cuenta de almacenamiento principal

Puede tener acceso directamente a los datos de la cuenta de almacenamiento principal. No hay necesidad de proporcionar las claves secretas. En Explorador de datos, haga clic con el botón derecho en un archivo y seleccione **Nuevo cuaderno** para ver un nuevo cuaderno con el extractor de datos generado automáticamente.

![data-to-cell](./media/apache-spark-development-using-notebooks/synapse-data-to-cell.png)

## <a name="save-notebooks"></a>Guardado de cuadernos

Puede guardar un único cuaderno o todos los cuadernos en el área de trabajo.

1. Para guardar los cambios realizados en un solo cuaderno, seleccione el botón **Publicar** en la barra de comandos del cuaderno.

   ![publish-notebook](./media/apache-spark-development-using-notebooks/synapse-publish-notebook.png)

2. Para guardar todos los cuadernos en el área de trabajo, seleccione el botón **Publicar todo** en la barra de comandos del área de trabajo. 

   ![publish-all](./media/apache-spark-development-using-notebooks/synapse-publish-all.png)

En las propiedades del cuaderno, puede configurar si incluir la salida de la celda al guardar.

   ![notebook-properties](./media/apache-spark-development-using-notebooks/synapse-notebook-properties.png)

## <a name="magic-commands"></a>Comandos magic
Puede usar los comandos magic de Jupyter que ya conoce en los cuadernos de Synapse. Revise la lista siguiente para ver los comandos magic disponibles actualmente. Indíquenos [sus casos de uso en GitHub](https://github.com/MicrosoftDocs/azure-docs/issues/new) para que podamos seguir generando más comandos magic para satisfacer sus necesidades.

> [!NOTE]
> Solo se admiten los comandos magic siguientes en una canalización de Synapse: %%pyspark, %%spark, %%csharp, %%sql. 
>
>

# <a name="classical-notebook"></a>[Cuaderno clásico](#tab/classical)

Comandos magic de línea disponibles: [%lsmagic](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-lsmagic), [%time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time), [%timeit](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit)

Comandos magic de celda disponibles: [%%time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time), [%%timeit](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit), [%%capture](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-capture), [%%writefile](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-writefile), [%%sql](#use-multiple-languages), [%%pyspark](#use-multiple-languages), [%%spark](#use-multiple-languages), [%%csharp](#use-multiple-languages),[%%configure](#spark-session-config-magic-command)



# <a name="preview-notebook"></a>[Versión preliminar del cuaderno](#tab/preview)

Comandos magic de línea disponibles: [%lsmagic](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-lsmagic), [%time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time), [%timeit](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit), [%history](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-history), [%run](#notebook-reference), [%load](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-load)

Comandos magic de celda disponibles: [%%time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time), [%%timeit](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit), [%%capture](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-capture), [%%writefile](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-writefile), [%%sql](#use-multiple-languages), [%%pyspark](#use-multiple-languages), [%%spark](#use-multiple-languages), [%%csharp](#use-multiple-languages), [%%html](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-html), [%%configure](#spark-session-config-magic-command)

--- 

## <a name="integrate-a-notebook"></a>Integración de un cuaderno

### <a name="add-a-notebook-to-a-pipeline"></a>Adición de un cuaderno a una canalización

Seleccione el botón **Agregar a la canalización** en la esquina superior derecha para agregar un cuaderno a una canalización existente o crear una canalización.

![Adición de un cuaderno a la canalización](./media/apache-spark-development-using-notebooks/add-to-pipeline.png)

### <a name="designate-a-parameters-cell"></a>Designación de una celda de parámetros

# <a name="classical-notebook"></a>[Cuaderno clásico](#tab/classical)

Para parametrizar el cuaderno, seleccione los puntos suspensivos (…) para acceder al menú de otras acciones de celda del extremo derecho. A continuación, seleccione **Toggle parameter cell** (Alternar celda de parámetros) para designar la celda como la celda de parámetros.

![toggle-parameter](./media/apache-spark-development-using-notebooks/toggle-parameter-cell.png)

# <a name="preview-notebook"></a>[Versión preliminar del cuaderno](#tab/preview)

Para parametrizar el cuaderno, seleccione los puntos suspensivos (...) para obtener acceso a **más comandos** en la barra de herramientas de la celda. A continuación, seleccione **Toggle parameter cell** (Alternar celda de parámetros) para designar la celda como la celda de parámetros.

![azure-notebook-toggle-parameter](./media/apache-spark-development-using-notebooks/azure-notebook-toggle-parameter-cell.png)

---

Azure Data Factory busca la celda de parámetros y la trata como valores predeterminados para los parámetros que se pasan en tiempo de ejecución. El motor de ejecución agregará una nueva celda debajo de la celda de parámetros con parámetros de entrada para sobrescribir los valores predeterminados. Cuando no se designa ninguna celda de parámetros, la celda insertada se insertará en la parte superior del cuaderno.


### <a name="assign-parameters-values-from-a-pipeline"></a>Asignación de valores de parámetros de una canalización

Una vez que haya creado un cuaderno con parámetros, podrá ejecutarlo desde una canalización con la actividad de cuadernos de Synapse. Después de agregar la actividad al lienzo de la canalización, podrá establecer los valores de los parámetros en la sección **Parámetros base** de la pestaña **Configuración**. 

![Asignación de un parámetro](./media/apache-spark-development-using-notebooks/assign-parameter.png)

Al asignar valores de parámetros, puede usar el [lenguaje de expresiones de canalización](../../data-factory/control-flow-expression-language-functions.md) o las [variables del sistema](../../data-factory/control-flow-system-variables.md).



## <a name="shortcut-keys"></a>Teclas de método abreviado

De forma similar a los cuadernos de Jupyter Notebook, los cuadernos de Synapse tienen una interfaz de usuario modal. El teclado realiza diferentes acciones en función del modo en que se encuentre la celda del cuaderno. Los cuadernos de Synapse admiten los dos modos siguientes para una celda de código determinada: modo de comando y modo de edición.

1. Una celda se encuentra en modo de comando cuando no hay ningún cursor de texto que le pida que escriba. Cuando una celda está en modo de comando, puede editar el cuaderno en su conjunto, pero no escribir en celdas individuales. Para ingresar al modo de comando, presione `ESC` o use el mouse para seleccionar fuera del área del editor de una celda.

   ![command-mode](./media/apache-spark-development-using-notebooks/synapse-command-mode-2.png)

2. El modo de edición se indica mediante un cursor de texto que le pide que escriba en el área del editor. Cuando una celda se encuentra en modo de edición, puede escribir en la celda. Para ingresar al modo de edición, presione `Enter` o use el mouse para seleccionar en el área del editor de una celda.
   
   ![edit-mode](./media/apache-spark-development-using-notebooks/synapse-edit-mode-2.png)

### <a name="shortcut-keys-under-command-mode"></a>Teclas de método abreviado en el modo de comando

# <a name="classical-notebook"></a>[Cuaderno clásico](#tab/classical)

Con los siguientes métodos abreviados de teclado, puede desplazarse y ejecutar código más fácilmente en los cuadernos de Synapse.

| Acción |Métodos abreviados de teclado de los cuadernos de Synapse  |
|--|--|
|Ejecutar la celda actual y seleccionar la que está a continuación | Mayús+Entrar |
|Ejecutar la celda actual e insertar una a continuación | Alt+Entrar |
|Seleccionar la celda anterior| Arriba |
|Seleccionar la celda siguiente| Bajar |
|Insertar una celda encima| A |
|Insertar una celda debajo| B |
|Extender las celdas seleccionadas encima| Mayús+Arriba |
|Extender las celdas seleccionadas debajo| Mayús+Abajo|
|Subir celda| CTRL+Alt+↑ |
|Bajar celda| CTRL+Alt+↓ |
|Eliminar celdas seleccionadas| D, D |
|Cambiar al modo de edición| Escriba |

# <a name="preview-notebook"></a>[Versión preliminar del cuaderno](#tab/preview)

| Acción |Métodos abreviados de teclado de los cuadernos de Synapse  |
|--|--|
|Ejecutar la celda actual y seleccionar la que está a continuación | Mayús+Entrar |
|Ejecutar la celda actual e insertar una a continuación | Alt+Entrar |
|Ejecutar celda actual| Ctrl+Entrar |
|Seleccionar la celda anterior| Arriba |
|Seleccionar la celda siguiente| Bajar |
|Seleccionar celda anterior| K |
|Seleccionar celda siguiente| J |
|Insertar una celda encima| A |
|Insertar una celda debajo| B |
|Eliminar celdas seleccionadas| Mayús+D |
|Cambiar al modo de edición| Escriba |

---

### <a name="shortcut-keys-under-edit-mode"></a>Teclas de método abreviado en el modo de edición


Con los siguientes métodos abreviados de teclado, puede desplazarse y ejecutar código más fácilmente en los cuadernos de Synapse cuando esté en modo de edición.

| Acción |Métodos abreviados de teclado de los cuadernos de Synapse  |
|--|--|
|Subir el cursor | Arriba |
|Bajar el cursor|Bajar|
|Deshacer|CTRL+Z|
|Rehacer|CTRL+Y|
|Comentar y quitar comentario|CTRL+/|
|Eliminar palabra anterior|CTRL+Retroceso|
|Eliminar palabra posterior|CTRL+Supr|
|Ir al inicio de la celda|CTRL+Inicio|
|Ir al final de la celda |CTRL+Fin|
|Ir una palabra a la izquierda|CTRL+Izquierda|
|Ir una palabra a la derecha|CTRL+Derecha|
|Seleccionar todo|CTRL+A|
|Aplicar sangría| Ctrl +]|
|Desaplicar sangría|CTRL+[|
|Cambiar al modo de comando| Esc |

---

## <a name="next-steps"></a>Pasos siguientes
- [Consulte los cuadernos de ejemplo de Synapse](https://github.com/Azure-Samples/Synapse/tree/master/Notebooks)
- [Inicio rápido: Creación de un grupo de Apache Spark en Azure Synapse Analytics mediante herramientas web](../quickstart-apache-spark-notebook.md)
- [Qué es Apache Spark en Azure Synapse Analytics](apache-spark-overview.md)
- [Uso de .NET para Apache Spark con Azure Synapse Analytics](spark-dotnet.md)
- [Documentación de .NET para Apache Spark](/dotnet/spark)
- [Azure Synapse Analytics](../index.yml)
