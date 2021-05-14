---
title: 'Tutorial: Herramientas Spark y Hive para VSCode (aplicación Spark)'
description: 'Tutorial: Uso de las herramientas Spark y Hive para VSCode a fin de desarrollar aplicaciones Spark escritas en Python y enviarlas a un grupo de Apache Spark sin servidor.'
services: synapse-analytics
author: jejiang
ms.author: jejiang
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: spark
ms.date: 09/03/2020
ms.openlocfilehash: 83d8d6498a5ec4906f7cdd7187ae88e4a2f45126
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108318034"
---
# <a name="tutorial-create-an-apache-spark-applications-with-vscode-using-a-synapse-workspace"></a>Tutorial: Creación de aplicaciones de Apache Spark con VSCode mediante un área de trabajo de Synapse

Obtenga información sobre cómo usar las herramientas Apache Spark y Apache Hive para Visual Studio Code. Use las herramientas para crear y enviar trabajos por lotes de Apache Hive, consultas de Hive interactivas y scripts de PySpark para Apache Spark. Primero, describiremos cómo instalar las herramientas Spark y Hive en Visual Studio Code. A continuación, veremos cómo enviar trabajos a las herramientas Spark y Hive.

Las herramientas Spark y Hive se pueden instalar en todas las plataformas compatibles con Visual Studio Code. Observe los siguientes requisitos previos para las distintas plataformas.

## <a name="prerequisites"></a>Requisitos previos

Los elementos siguientes son necesarios para completar los pasos indicados en este artículo:

- Un grupo de Apache Spark sin servidor. Para crear un grupo de Apache Spark sin servidor, consulte [Creación de grupos de Apache Spark mediante Azure Portal](../../synapse-analytics/quickstart-create-apache-spark-pool-portal.md).
- [Visual Studio Code](https://code.visualstudio.com/).
- [Mono](https://www.mono-project.com/docs/getting-started/install/). Mono solo es obligatorio para Linux y macOS.
- [Un entorno interactivo de PySpark para Visual Studio Code](../../hdinsight/set-up-pyspark-interactive-environment.md).
- Un directorio local. En este artículo se usa **C:\HD\Synaseexample**.

## <a name="install-spark--hive-tools"></a>Instalar Spark & Hive Tools

Después de completar los requisitos previos, puede instalar Spark & Hive Tools para Visual Studio Code siguiendo estos pasos:

1. Abra Visual Studio Code.

2. En la barra de menús, vaya a **Ver** > **Extensiones**.

3. En el cuadro de búsqueda, escriba **Spark & Hive**.

4. Seleccione **Spark & Hive Tools** en los resultados de búsqueda y seleccione **Instalar**:

     ![Instalación de Spark y Hive para Visual Studio Code Python](./media/vscode-tool-synapse/install-hdInsight-plugin.png)

5. Seleccione **Recargar** cuando sea necesario.

## <a name="open-a-work-folder"></a>Apertura de una carpeta de trabajo

Para abrir una carpeta de trabajo y crear un archivo en Visual Studio Code, siga estos pasos:

1. En la barra de menús, vaya a **Archivo** > **Abrir carpeta...**  > **C:\HD\Synaseexample** y, a continuación, haga clic en el botón **Seleccionar carpeta**. La carpeta se mostrará en la vista **Explorador** de la parte izquierda.

2. En la vista de **Explorer**, seleccione la carpeta **Synaseexample** y, a continuación, seleccione el icono **Nuevo archivo** situado junto a la carpeta de trabajo:

     ![Icono de archivo nuevo de Visual Studio Code](./media/vscode-tool-synapse/visual-studio-code-new-file.png)

3. Asigne al nuevo archivo un nombre con la extensión de archivo `.py` (script de Spark). En este ejemplo, se usa **HelloWorld.py**.

## <a name="connect-to-your-spark-pools"></a>Conexión a los grupos de Spark

Inicie sesión en la suscripción de Azure para conectarse a los grupos de Spark.

### <a name="sign-in-to-your-azure-subscription"></a>Inicie sesión en la suscripción de Azure

Siga estos pasos para conectarse a Azure:

1. En la barra de menús, vaya a **Ver** > **Paleta de comandos...** y escriba **Azure: Sign In**:

     ![Usar las herramientas Spark y Hive para el inicio de sesión de Visual Studio Code](./media/vscode-tool-synapse/hdinsight-for-vscode-extension-login.png)

2. Siga las instrucciones para iniciar sesión en Azure. Una vez establecida la conexión, el nombre de la cuenta de Azure se muestra en la barra de estado en la parte inferior de la ventana de Visual Studio Code.

## <a name="set-the-default-spark-pool"></a>Establecimiento del grupo de Spark predeterminado

1. Vuelva a abrir la carpeta **Synaseexample** que se explicó [anteriormente](#open-a-work-folder), si está cerrada.  

2. Seleccione el archivo **HelloWorld.py** que creó [anteriormente](#open-a-work-folder). Se abre en el editor de scripts.

3. Haga clic con el botón derecho en el editor de scripts y, a continuación, seleccione **Synapse: Establecimiento del grupo de Spark predeterminado**.  

4. [Conéctese](#connect-to-your-spark-pools) a su cuenta de Azure si no lo ha hecho aún.

5. Seleccione un grupo de Spark como predeterminado para el archivo de script actual. 

6. Use **Synapse: PySpark Interactive** para enviar este archivo. Y las herramientas actualizan automáticamente el archivo de configuración **.VSCode\settings.json**:

     ![Establecer la configuración del clúster predeterminado](./media/vscode-tool-synapse/set-default-cluster-configuration.png)

## <a name="submit-interactive-synapse-pyspark-queries-to-spark-pool"></a>Envío de consultas interactivas de Synapse PySpark al grupo de Spark

Los usuarios pueden interactuar con Synapse PySpark en el grupo de Spark de las maneras siguientes:

### <a name="using-the-synapse-pyspark-interactive-command-in-py-file"></a>Con el comando de Synapse PySpark Interactive en el archivo PY
Para usar el comando de PySpark Interactive con el fin de enviar las consultas, siga estos pasos:

1. Vuelva a abrir la carpeta **Synaseexample** que se explicó [anteriormente](#open-a-work-folder), si está cerrada.  

2. Cree un nuevo archivo **HelloWorld.py** siguiendo los pasos [anteriores](#open-a-work-folder).

3. Copie y pegue el código siguiente en el archivo de script:

```python
import sys
from operator import add
from pyspark.sql import SparkSession, Row
 
spark = SparkSession\
 .builder\
 .appName("PythonWordCount")\
 .getOrCreate()
 
data = [Row(col1='pyspark and spark', col2=1), Row(col1='pyspark', col2=2), Row(col1='spark vs hadoop', col2=2), Row(col1='spark', col2=2), Row(col1='hadoop', col2=2)]
df = spark.createDataFrame(data)
lines = df.rdd.map(lambda r: r[0])
 
counters = lines.flatMap(lambda x: x.split(' ')) \
 .map(lambda x: (x, 1)) \
 .reduceByKey(add)
 
output = counters.collect()
sortedCollection = sorted(output, key = lambda r: r[1], reverse = True)
 
for (word, count) in sortedCollection:
 print("%s: %i" % (word, count))
```

4. El mensaje para instalar el kernel de PySpark/Synapse Pyspark se muestra en la esquina inferior derecha de la ventana. Puede hacer clic en el botón **Instalar** para continuar con las instalaciones de Pyspark o Synaps PySpark, o bien hacer clic en el botón **Omitir** para omitir este paso.

     ![instalación del kernel de pyspark](./media/vscode-tool-synapse/install-the-pyspark-kernel.png)

5. Si tiene que instalarlo más adelante, puede ir a **File** (Archivo)  > **Preference** (Preferencia)  > **Settings** (Configuración) y, después, desactivar **Hdinsight: Enable Skip Pyspark Installation** (Habilitar Omitir la instalación de Pyspark) en las opciones. 
    
     ![Habilitar Omitir la instalación de Pyspark](./media/vscode-tool-synapse/enable-skip-pyspark-installation.png)

6. Si la instalación se realiza correctamente en el paso 4, el cuadro de mensaje "PySpark/Synapse Pyspark installed successfully" (PySpark/Synapse Pyspark se instaló correctamente) se muestra en la esquina inferior derecha de la ventana. Haga clic en botón **Reload** (Recargar) para volver a cargar la ventana.

     ![pyspark se ha instalado correctamente](./media/vscode-tool-synapse/pyspark-kernel-installed-successfully.png)

7. En la barra de menús, vaya a **View** (Ver)  > **Command Palette** (Paleta de comandos) o use el método abreviado de teclado **Mayús + Ctrl + P** y escriba **Python: Seleccione Interpreter (Intérprete) para iniciar el servidor de Jupyter**.

     ![Seleccionar Interpreter para iniciar el servidor de Jupyter](./media/vscode-tool-synapse/select-interpreter-to-start-jupyter-server.png)

8. Seleccione la opción de Python que aparece a continuación.

     ![Elija la opción siguiente](./media/vscode-tool-synapse/choose-the-below-option.png)
    
9. En la barra de menús, vaya a **View**(Ver) > **Command Palette** (Paleta de comandos) o use el método abreviado de teclado **Mayús + Ctrl + P** y escriba **Desarrollador: Recargar ventana**.

     ![Recargar ventana](./media/vscode-tool-synapse/reload-window.png)

10. [Conéctese](#connect-to-your-spark-pools) a su cuenta de Azure si no lo ha hecho aún.

11. Seleccione todo el código, haga clic con el botón derecho en el editor de scripts y seleccione **Synapse: Pyspark Interactive** para enviar la consulta. 

     ![menú contextual de PySpark Interactive](./media/vscode-tool-synapse/pyspark-interactive-right-click.png)

12. Seleccione el grupo de Spark, si no ha especificado un grupo de Spark predeterminado. En un momento, los resultados de **Python Interactive** se mostrarán en una pestaña nueva. Haga clic en PySpark para cambiar el kernel a **Synapse PySpark** y, a continuación, vuelva a enviar el código seleccionado y este se ejecutará correctamente. Las herramientas también permiten enviar un bloque de código en lugar del archivo de script completo mediante el menú contextual:

     ![interactiva](./media/vscode-tool-synapse/pyspark-interactive-python-interactive-window.png)

### <a name="perform-interactive-query-in-py-file-using-a--comment"></a>Consulta interactiva en el archivo PY con un comentario #%%

1. Agregue **#%%** antes del código para obtener una experiencia de cuaderno.

     ![agregue #%%](./media/vscode-tool-synapse/run-cell.png)

2. Haga clic en **Run Cell** (Ejecutar celda). En un momento, los resultados de Python Interactive se mostrarán en una pestaña nueva. Haga clic en PySpark para cambiar el kernel a **Synapse PySpark** y, a continuación, vuelva a hacer clic en **Ejecutar celda** y el código se ejecutará correctamente. 

     ![ejecución de los resultados de celda](./media/vscode-tool-synapse/run-cell-get-results.png)

## <a name="leverage-ipynb-support-from-python-extension"></a>Aprovechar la compatibilidad con IPYNB de la extensión de Python

1. Puede crear un comando de Jupyter Notebook en la paleta de comandos o mediante la creación de un nuevo archivo. ipynb en el área de trabajo. Para más información, consulte [Trabajo con cuadernos de Jupyter Notebook en Visual Studio Code](https://code.visualstudio.com/docs/python/jupyter-support).

2. Haga clic en el botón **Run cell** (Ejecutar celda), siga las indicaciones para **Set the default spark pool** (Establecer el grupo de Spark predeterminado) (se recomienda encarecidamente establecer el grupo o clúster predeterminado cada vez antes de abrir un cuaderno) y, a continuación, **Reload** (Recargar).

     ![Establecer el grupo de Spark predeterminado y volver a cargarlo](./media/vscode-tool-synapse/set-the-default-spark-pool-and-reload.png)

3. Haga clic en PySpark para cambiar el kernel a **Synapse Pyspark** y, después, haga clic en **Run Cell** (Ejecutar celda); al cabo de un rato, se mostrará el resultado.

     ![ejecución de los resultados de ipynb](./media/vscode-tool-synapse/run-ipynb-file-results.png)


> [!NOTE]
>
>1. Las versiones de ms-python a partir de 2020.5.78807 no se admiten en esta extensión. Esto es un [problema conocido](#known-issues).
>  
>2. Cambie al kernel de Synapse Pyspark. Se recomienda deshabilitar la configuración automática en Azure Portal. En caso contrario, es posible que se tarde mucho tiempo en reactivar el clúster y establecer el kernel de Synapse para el primer uso. 
>
>    ![configuración automática](./media/vscode-tool-synapse/auto-settings.png)

## <a name="submit-pyspark-batch-job-to-spark-pool"></a>Envío de un trabajo por lotes de PySpark al grupo de Spark

1. Vuelva a abrir la carpeta **Synaseexample** que explicó [anteriormente](#open-a-work-folder), si está cerrada.  

2. Cree un nuevo archivo **BatchFile.py** siguiendo los pasos [anteriores](#open-a-work-folder).

3. Copie y pegue el código siguiente en el archivo de script:

    ```python
    from __future__ import print_function
    import sys
    from operator import add
    from pyspark.sql import SparkSession
    if __name__ == "__main__":
        spark = SparkSession\
            .builder\
            .appName("PythonWordCount")\
            .getOrCreate()
    
        lines = spark.read.text('/HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv').rdd.map(lambda r: r[0])
        counts = lines.flatMap(lambda x: x.split(' '))\
                    .map(lambda x: (x, 1))\
                    .reduceByKey(add)
        output = counts.collect()
        for (word, count) in output:
            print("%s: %i" % (word, count))
        spark.stop()
    ```

4. [Conéctese](#connect-to-your-spark-pools) a su cuenta de Azure si no lo ha hecho aún.

5. Haga clic con el botón derecho en el editor de scripts y, a continuación, seleccione **Synapse: PySpark Batch** (Synapse: lote de PySpark).

6. Seleccione un grupo de Spark para enviar el trabajo de PySpark para:

     ![Enviar la salida del resultado del trabajo de Python](./media/vscode-tool-synapse/submit-pythonjob-result.png)

Después de enviar un trabajo por lotes al grupo de Spark, los registros de envío aparecen en la ventana **SALIDA** en Visual Studio Code. También se muestran las direcciones URL de la **interfaz de usuario de Spark** y la **interfaz de usuario de la aplicación de trabajo de Spark**. Para realizar un seguimiento del estado del trabajo, puede abrir la URL en un explorador web.

## <a name="access-and-manage-synapse-workspace"></a>Acceso y administración del área de trabajo de Synapse

Puede realizar diferentes operaciones en el Explorador de Azure en las herramientas Spark y Hive para VSCode. En el Explorador de Azure.

![imagen de Azure](./media/vscode-tool-synapse/azure.png)

### <a name="launch-workspace"></a>Inicio del área de trabajo

1. En el Explorador de Azure, vaya a **SYNAPSE**, expándalo y muestre la lista de suscripciones de Synapse.

     ![Explorador de Synapse](./media/vscode-tool-synapse/synapse-explorer.png)

2. Haga clic en el área de trabajo Subscription of Synapse (Suscripción de Synapse), expándala y muestre la lista de áreas de trabajo.

3. Haga clic con el botón secundario en un área de trabajo y, a continuación, seleccione **View Apache Spark applications** (Ver aplicaciones de Apache Spark). Se abrirá la página de la aplicación Apache Spark en el sitio web de Synapse Studio.

     ![Hacer clic con el botón derecho en el área de trabajo](./media/vscode-tool-synapse/right-click-on-workspace.png)

     ![Aplicación Synapse Studio](./media/vscode-tool-synapse/synapse-studio-application.png)

4. Expanda un área de trabajo y se mostrarán **Almacenamiento predeterminado** y **Grupos de Apache Spark**.

5. Haga clic con el botón derecho en **Almacenamiento predeterminado** y se mostrarán **Copiar ruta de acceso completa** y **Abrir en Synapse Studio**. 

     ![Hacer clic con el botón derecho en Almacenamiento predeterminado](./media/vscode-tool-synapse/right-click-on-default-storage.png)

     - Haga clic en **Copiar ruta de acceso completa** y se copiará la dirección URL principal de la cuenta de ADLS Gen2, que podrá pegar donde necesite.

     - Haga clic en **Abrir en Synapse Studio** y la cuenta de almacenamiento principal se abrirá en Synapse Studio.

     ![Almacenamiento predeterminado en Synapse Studio](./media/vscode-tool-synapse/default-storage-in-synapse-studio.png)

6. Expanda **Almacenamiento predeterminado** y se mostrará la cuenta de almacenamiento principal.

7. Expanda **Grupos de Apache Spark** y se mostrarán todos los grupos de Spark en el área de trabajo.

     ![Expandir grupo de almacenamiento](./media/vscode-tool-synapse/expand-storage-pool.png)


## <a name="known-issues"></a>Problemas conocidos

### <a name="ms-python-2020578807-version-is-not-supported-on-this-extention"></a>No se admiten las versiones de ms-python a partir de 2020.5.78807, incluida, en esta extensión 

"No se pudo conectar con el cuaderno de Jupyter Notebook" es un problema conocido de las versiones de Python a partir de la 2020.5.78807, incluida. Se recomienda que los usuarios usen la versión **[2020.4.76186](https://github.com/microsoft/vscode-python/releases/download/2020.4.76186/ms-python-release.vsix)** de ms-python para evitar este problema.

![Problemas conocidos](./media/vscode-tool-synapse/known-issue.png)


## <a name="next-steps"></a>Pasos siguientes

- [Azure Synapse Analytics](../overview-what-is.md)
- [Creación de un grupo de Apache Spark para un área de trabajo de Azure Synapse Analytics](../../synapse-analytics/quickstart-create-apache-spark-pool-studio.md)
