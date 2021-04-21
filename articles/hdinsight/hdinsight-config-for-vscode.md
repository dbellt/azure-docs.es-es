---
title: Referencia de configuración de Azure HDInsight
description: Introduzca la configuración de la extensión de Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.date: 04/07/2021
ms.custom: devx-track-python
ms.openlocfilehash: a9a444c2557ea17114123cbd5084cbbd9fe6dac6
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107259935"
---
# <a name="azure-hdinsight-configuration-settings-reference"></a>Referencia de configuración de Azure HDInsight

La extensión de las herramientas de Spark y Hive para Visual Studio Code es altamente configurable. En esta página se describe la configuración clave con la que puede trabajar.  

Para información general sobre cómo trabajar con la configuración en VS Code, consulte [Configuración del usuario y el área de trabajo](https://code.visualstudio.com/docs/getstarted/settings) y [Referencia de variables](https://code.visualstudio.com/docs/editor/variables-reference) para información sobre la compatibilidad con variables predefinidas.

## <a name="open-the-azure-hdinsight-configuration"></a>Apertura de la configuración de Azure HDInsight

1. Abra primero una carpeta para crear la configuración del área de trabajo.
2. Presione **Ctrl + Mayús + P** o vaya a **Ver** -> **Paleta de comandos...** para mostrar todos los comandos.
3. Busque **Set Configuration** (Establecer configuración).
4. Expanda **Extensiones** en el directorio izquierdo y seleccione **Configuración de HDInsight**.

 ![imagen de configuración de hdi](./media/HDInsight-config-for-vscode/HDInsight-config-for-vscode.png)

## <a name="general-settings"></a>Configuración general   

|  Propiedad   | Valor predeterminado | Descripción   |
| ----- | ----- |----- |
| HDInsight: Azure Environment | Azure | Entorno de Azure |
| HDInsight: Disable Open Survey Link | Activada | Habilitar o deshabilitar la apertura de la encuesta de HDInsight. |
| HDInsight: Enable Skip Pyspark Installation | No activado | Habilitar o deshabilitar la omisión de la instalación de pyspark. |
| HDInsight: Login Tips Enable | No activado | Cuando se activa esta opción, se muestra un mensaje al iniciar sesión en Azure. |
| HDInsight: Previous Extension Version | Mostrar el número de versión de la extensión actual | Mostrar la versión de extensión anterior.|
| HDInsight: Results Font Family | -apple-system,BlinkMacSystemFont,Segoe WPC,Segoe UI,HelveticaNeue-Light,Ubuntu,Droid Sans,sans-serif | Establecer la familia de fuentes para la cuadrícula de resultados; dejar en blanco para usar la fuente del editor |
| HDInsight: Results Font Size | 13 |Establecer el tamaño de la fuente para la cuadrícula de resultados; dejar en blanco para usar el tamaño del editor |
| HDInsight Cluster: Linked Cluster | -- | Direcciones URL de los clústeres vinculados. También puede editar el archivo JSON que se va a establecer. |
| HDInsight Hive: Apply Localization | No activado | [Opcional] Opciones de configuración para localizar en la configuración regional establecida de VSCode (debe reiniciarse VSCode para que la configuración entre en vigencia).|
| HDInsight Hive: Copy Include Headers | No activado | [Opcional] Opción de configuración para copiar los resultados de la vista de resultados. |
| HDInsight Hive: Copy Remove New Line | Activada | [Opcional] Opciones de configuración para copiar los resultados de varias líneas de la vista de resultados. |
| HDInsight Hive › Format: Align Column Definitions In Columns | No activado | Indica si la definición de columna debe alinearse |
| HDInsight Hive › Format: Datatype Casing | ninguno | Indica si los tipos de datos deben formatearse como MAYÚSCULAS, minúsculas o nada (sin formato). |
| HDInsight Hive › Format: Keyword Casing | ninguno | Indica si las palabras clave deben formatearse como MAYÚSCULAS, minúsculas o nada (sin formato). |
| HDInsight Hive › Format: Place Commas Before Next Statement | No activado | Indica si las comas deben colocarse al principio de cada instrucción de una lista por ejemplo, ", micolumna2" en lugar de al final, por ejemplo, "micolumna1,".|
| HDInsight Hive › Format: Place Select Statement References On New Line | No activado | ¿Deben las referencias a objetos de una instrucción select dividirse en líneas independientes? Por ejemplo, para "SELECT C1, C2 FROM T1", tanto C1 como C2 estarán en líneas independientes.
| HDInsight Hive: Log Debug Info | No activado | [Opcional] Registre la salida de depuración en la consola de VS Code (Ayuda -> Alternar herramientas de desarrollo). 
| HDInsight Hive: Messages Default Open | Activada | True para que el panel de mensajes se abra de forma predeterminada; false para que esté cerrado.|
| HDInsight Hive: Results Font Family | -apple-system,BlinkMacSystemFont,Segoe WPC,Segoe UI,HelveticaNeue-Light,Ubuntu,Droid Sans,sans-serif | Establecer la familia de fuentes para la cuadrícula de resultados; dejar en blanco para usar la fuente del editor |
| HDInsight Hive: Results Font Size | 13 | Establecer el tamaño de la fuente para la cuadrícula de resultados; dejar en blanco para usar el tamaño del editor |
| HDInsight Hive › Save As Csv: Include Headers | Activada | [Opcional] Si se establece en true, los encabezados de columna se incluyen al guardar los resultados como CSV. |
| HDInsight Hive: Shortcuts | -- | Accesos directos relacionados con la ventana de resultados. |
| HDInsight Hive: Show Batch Time| No activado | [Opcional] Indica si se muestra el tiempo de ejecución para los lotes individuales. |
| HDInsight Hive: Split Pane Selection | Siguiente | [Opcional] Opciones de configuración en las que deben abrirse los paneles de resultados nuevos de columna |
| HDInsight Job Submission: Cluster Conf | -- | Configuración de clúster |
| HDInsight Job Submission: Livy Conf | -- | Configuración de Livy. POST/lotes. |
| HDInsight Jupyter: Append Results| Activada | Si se anexan los resultados a la ventana de resultados, si no se borra y se muestra. |
| HDInsight Jupyter: Languages | -- | Configuración predeterminada por idioma. |
| HDInsight Jupyter › Log: Verbose | No activado | Habilitar el registro detallado. |
| HDInsight Jupyter › Notebook: Startup Args | Se pueden agregar elementos | Argumentos de línea de comandos de "jupyter notebook". Cada argumento es un elemento independiente de la matriz. Para una lista completa, escriba "jupyter notebook--help" en una ventana del terminal. |
| HDInsight Jupyter › Notebook: Startup Folder | ${workspaceRoot} |-- |
| HDInsight Jupyter: Python Extension Enabled | Activada | Use Python-Interactive-Window de la extensión ms-python al enviar trabajos de pySpark interactivos. En el resto de casos, use nuestra propia ventana de Jupyter. |
| HDInsight Spark.NET: 7z | C:\Archivos de programa\7-Zip | <Ruta de acceso a 7z.exe> |
| HDInsight Spark.NET: HADOOP_HOME | D:\winutils | <Ruta de acceso a bin\winutils.exe> solo sistema operativo Windows. |
| HDInsight Spark.NET: JAVA_HOME | C:\Archivos de programa\Java\jdk1.8.0_201\ | Ruta de acceso al inicio de Java.|
| HDInsight Spark.NET: SCALA_HOME | C:\Archivos de programa (x86)\scala\ | Ruta de acceso al inicio de Scala. |
| HDInsight Spark.NET: SPARK_HOME | D:\spark-2.3.3-bin-hadoop2.7\ | Ruta de acceso al inicio de Spark. |
| Hive: Persist Query Result Tabs | No activado | Hive PersistQueryResultTabs |
| Hive: Split Pane Selection | Siguiente | [Opcional] Opciones de configuración en las que deben abrirse los paneles de resultados nuevos de columna |
| Hive Interactive: Copy Executable Folder | No activado | Copiar la carpeta del entorno de tiempo de ejecución del servicio interactivo de Hive en la carpeta tmp del usuario. |
| Hql Interactive Server: Wrapper Port | 13424 | Puerto de servicio interactivo de Hive. |
| Hql Language Server: Language Wrapper Port | 12342 | Puerto de servicio de lenguaje de Hive al que escuchan los servidores. |
| Hql Language Server: Max Number Of Problems | 100 | Controla el número máximo de problemas generados por el servidor. |
| Synapse Spark Compute: Synapse Spark Compute Azure Environment | en blanco | Entorno de Azure de proceso de Spark en Synapse. |
| Synapse Spark pool Job Submission: Livy Conf | -- | Configuración de Livy. POST/lotes.
| Synapse Spark pool Job Submission: Synapse Spark Pool Cluster Conf | -- | Configuración del grupo de Spark en Synapse |


## <a name="next-steps"></a>Pasos siguientes

- Para información sobre Azure HDInsight para VSCode, consulte [Spark y Hive para Visual Studio Code Tools](https://docs.microsoft.com/sql/big-data-cluster/spark-hive-tools-vscode).
- Para ver un vídeo de demostración del uso de Spark y Hive para Visual Studio Code, consulte [Spark y Hive para Visual Studio Code](https://go.microsoft.com/fwlink/?linkid=858706).