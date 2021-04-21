---
title: Solución de errores de instalación de biblioteca
description: En este tutorial se proporciona información general sobre cómo solucionar errores de instalación de la biblioteca.
services: synapse-analytics
author: midesa
ms.author: midesa
ms.service: synapse-analytics
ms.subservice: spark
ms.topic: conceptual
ms.date: 01/04/2021
ms.openlocfilehash: 006abf62c605c2ca34fd1adeadee8e29ae0fb8fb
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107588316"
---
# <a name="troubleshoot-library-installation-errors"></a>Solución de errores de instalación de biblioteca 
Para que el código de terceros o el código compilado localmente esté disponible para las aplicaciones, puede instalar una biblioteca en uno de los grupos de Apache Spark sin servidor. Los paquetes que aparecen en el archivo requirements.txt se descargan de PyPi cuando se inicia el grupo. Este archivo de requisitos se usa cada vez que se crea una instancia de Spark desde ese grupo de Spark. Una vez instalada una biblioteca para un grupo de Spark, está disponible para todas las sesiones que usan el mismo grupo. 

En algunos casos, puede encontrar que una biblioteca no aparece en el grupo de Apache Spark. Esto suele ocurrir si hay un error en el archivo requirements.txt proporcionado o en las bibliotecas especificadas. Cuando se produce un error en el proceso de instalación de la biblioteca, el grupo de Apache Spark se revertirá a las bibliotecas especificadas en el runtime base de Azure Synapse.

El objetivo de este documento es proporcionar información sobre problemas comunes y ayudarlo a depurar los errores de instalación de la biblioteca.

## <a name="force-update-your-apache-spark-pool"></a>Forzar la actualización del grupo de Apache Spark
Al actualizar las bibliotecas del grupo de Apache Spark, estos cambios se seleccionarán una vez que el grupo se reinicia. Si tiene trabajos activos, estos trabajos seguirán ejecutándose en la versión original del grupo de Spark.

Para forzar la aplicación de los cambios, seleccione la opción para **forzar la nueva configuración**. Esta configuración terminará todas las sesiones actuales del grupo de Spark seleccionado. Una vez finalizadas las sesiones, tendrá que esperar a que el grupo se reinicie. 

![Adición de bibliotecas de Python](./media/apache-spark-azure-portal-add-libraries/update-libraries.png "Adición de bibliotecas de Python")

## <a name="track-installation-progress"></a>Seguimiento del progreso de la instalación
Cada vez que se actualiza un grupo con un nuevo conjunto de bibliotecas, se inicia un trabajo de Spark reservado para el sistema. Este trabajo de Spark ayuda a supervisar el estado de la instalación de las bibliotecas. Si se produce un error en la instalación debido a conflictos entre bibliotecas u otros problemas, el grupo de Spark volverá a su estado anterior o predeterminado. 

Además, los usuarios pueden inspeccionar los registros de instalación para identificar conflictos de dependencia o ver qué bibliotecas se instalaron durante la actualización del grupo.

Para consultar estos registros:
1. Vaya a la lista de aplicaciones de Spark en la pestaña **Supervisar**. 
2. Seleccione el trabajo del sistema de la aplicación de Spark que corresponde a la actualización del grupo. Estos trabajos del sistema se ejecutan con el nombre *SystemReservedJob-LibraryManagement*.
   ![Captura de pantalla que resalta el trabajo de biblioteca reservado para el sistema.](./media/apache-spark-azure-portal-add-libraries/system-reserved-library-job.png "Visualización del trabajo de biblioteca del sistema")
3. Cambie para ver los registros del **controlador** y de **stdout**. 
4. En los resultados, verá los registros relacionados con la instalación de los paquetes.
    ![Captura de pantalla que resalta los resultados del trabajo de biblioteca reservado para el sistema.](./media/apache-spark-azure-portal-add-libraries/system-reserved-library-job-results.png "Visualización del progreso del trabajo de biblioteca del sistema")

## <a name="validate-your-permissions"></a>Validación de los permisos
Para instalar y actualizar las bibliotecas, debe tener los permisos de tipo **Colaborador de datos de Storage Blob** o **Propietario de datos de Storage Blob** en la cuenta de almacenamiento principal de Azure Data Lake Storage Gen2 que está vinculada con el área de trabajo de Azure Synapse Analytics.

Para validar que tiene estos permisos, puede ejecutar el código siguiente:

```python
from pyspark.sql.types import StructType,StructField, StringType, IntegerType
data2 = [("James","Smith","Joe","4355","M",3000),
    ("Michael","Rose","Edward","40288","F",4000)
  ]

schema = StructType([ \
    StructField("firstname",StringType(),True), \
    StructField("middlename",StringType(),True), \
    StructField("lastname",StringType(),True), \
    StructField("id", StringType(), True), \
    StructField("gender", StringType(), True), \
    StructField("salary", IntegerType(), True) \
  ])
 
df = spark.createDataFrame(data=data2,schema=schema)

df.write.csv("abfss://<<ENTER NAME OF FILE SYSTEM>>@<<ENTER NAME OF PRIMARY STORAGE ACCOUNT>>.dfs.core.windows.net/validate_permissions.csv")

```
Si recibe un error, es probable que le falten los permisos necesarios. Para saber cómo obtener los permisos necesarios, consulte este documento: [Asignación de los permisos Colaborador de datos de Storage Blob o Propietario de datos de Storage Blob](../../storage/common/storage-auth-aad-rbac-portal.md#assign-an-azure-built-in-role).

Además, si ejecuta una canalización, el MSI del área de trabajo también debe tener los permisos Propietario de datos de Storage Blob o Colaborador de datos de Storage Blob. Para información sobre cómo conceder este permiso a su identidad de área de trabajo, visite: [Concesión de permisos a una identidad administrada de área de trabajo](../security/how-to-grant-workspace-managed-identity-permissions.md).

## <a name="check-the-environment-configuration-file"></a>Revisión del archivo de configuración del entorno
Se puede usar un archivo de configuración de entorno para actualizar el entorno de Conda. [Aquí](./apache-spark-manage-python-packages.md) se enumeran los formatos de archivo admitidos para la administración de grupos de Python.

Es importante tener en cuenta las restricciones siguientes:
   -  El contenido del archivo de requisitos no debe incluir líneas o caracteres en blanco adicionales. 
   -  El [entorno de ejecución de Synapse](apache-spark-version-support.md) incluye un conjunto de bibliotecas instalado previamente en cada grupo de Apache Spark sin servidor. Los paquetes instalados previamente en el entorno de ejecución base no pueden quitar ni desinstalar.
   -  No se admite la modificación de la versión de PySpark, Python, Scala/Java, .NET o Spark.
   -  Las bibliotecas de ámbito de sesión de Python solo aceptan archivos con la extensión YML.

## <a name="validate-wheel-files"></a>Validación de los archivos Wheel
Los grupos de Apache Spark sin servidor de Synapse se basan en la distribución de Linux. Al descargar e instalar archivos Wheel directamente de PyPI, asegúrese de seleccionar la versión que se basa en Linux y que se ejecuta en la misma versión de Python que el grupo de Spark.

>[!IMPORTANT]
>Los paquetes personalizados se pueden agregar o modificar entre sesiones. Pero deberá esperar a que el grupo y la sesión se reinicien para ver el paquete actualizado.

## <a name="check-for-dependency-conflicts"></a>Comprobación de conflictos de dependencia
 En general, la resolución de dependencias de Python puede ser difícil de administrar. Para ayudar a depurar localmente los conflictos de dependencia, puede crear su propio entorno virtual basado en el entorno de ejecución de Synapse y validar los cambios.

Para volver a crear el entorno y validar las actualizaciones, haga lo siguiente:
 1. [Descargue](https://github.com/Azure-Samples/Synapse/blob/main/Spark/Python/base_environment.yml) la plantilla para volver a crear el entorno de ejecución de Synapse de manera local. Puede haber pequeñas diferencias entre la plantilla y el entorno de Synapse real.
   
 2. Use [estas instrucciones](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#activating-an-environment) para crear un entorno virtual. Este entorno le permite crear una instalación de Python aislada con la lista de bibliotecas especificada. 
    
    ```
    conda myenv create -f environment.yml
    conda activate myenv
    ```
   
 3. Use ``pip install -r <provide your req.txt file>`` para actualizar el entorno virtual con los paquetes especificados. Si se produce un error en la instalación, puede haber un conflicto entre lo que se instaló previamente en el entorno de ejecución base de Synapse y lo que se especifica en el archivo de requisitos proporcionado. Es necesario resolver estos conflictos de dependencia para obtener las bibliotecas actualizadas en el grupo de Apache Spark sin servidor.

>[!IMPORTANT]
>Pueden surgir problemas al usar PIP y Conda juntos. Al combinar PIP y Conda, es mejor seguir estos [procedimientos recomendados](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#activating-an-environment).

## <a name="next-steps"></a>Pasos siguientes
- Visualización de las bibliotecas predeterminadas: [Compatibilidad de las versiones de Azure Spark](apache-spark-version-support.md)
