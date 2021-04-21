---
title: Administración de bibliotecas de Python para Apache Spark
description: Obtenga información sobre cómo agregar y administrar bibliotecas de Python para Apache Spark en Azure Synapse Analytics.
services: synapse-analytics
author: midesa
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 02/26/2020
ms.author: midesa
ms.reviewer: jrasnick
ms.subservice: spark
ms.openlocfilehash: 1d64233fc477ec25f91bb73c744b10210571df41
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107588350"
---
# <a name="manage-python-libraries-for-apache-spark-in-azure-synapse-analytics"></a>Administración de bibliotecas de Python para Apache Spark en Azure Synapse Analytics

Las bibliotecas proporcionan código reutilizable que se puede incluir en los programas o proyectos. 

Es posible que tenga que actualizar el entorno de grupo sin servidor de Apache Spark por varios motivos. Por ejemplo:
- Si se acaba de publicar una nueva versión de una de sus dependencias principales.
- Si necesita un paquete adicional para entrenar su modelo de aprendizaje automático o preparar sus datos.
- Si ha encontrado un paquete mejor y ya no necesita el paquete anterior.

A fin de que el código de terceros o de compilación local esté disponible para las aplicaciones, puede instalar una biblioteca en uno de los grupos de Apache Spark sin servidor o en una sesión de cuaderno. En este artículo, se explica cómo puede administrar las bibliotecas de Python en el grupo de Apache Spark sin servidor.

## <a name="default-installation"></a>Instalación predeterminada
Apache Spark en Azure Synapse Analytics tiene un conjunto completo de bibliotecas para tareas comunes de ingeniería de datos, preparación de datos, aprendizaje automático y visualización de datos. Puede encontrar la lista de bibliotecas completas en [Compatibilidad con las versiones de Apache Spark](apache-spark-version-support.md). 

Cuando se inicie una instancia de Spark, estas bibliotecas se incluirán automáticamente. Se pueden agregar paquetes adicionales de compilación personalizada y de Python en el nivel de sesión y de grupo de Spark.

## <a name="pool-libraries"></a>Bibliotecas de grupos
Una vez que haya identificado las bibliotecas de Python que le gustaría usar para la aplicación Spark, puede instalarlas en un grupo de Spark. Las bibliotecas de nivel de grupo están disponibles para todos los cuadernos y todos los trabajos que se ejecutan en el grupo.

Hay dos maneras principales de instalar una biblioteca en un clúster:
-  Instalar una biblioteca de área de trabajo que se haya cargado como un paquete de área de trabajo.
-  Proporcionar un archivo *requirements.txt* o un archivo *environment.yml de Conda* de especificación de entorno para instalar paquetes desde repositorios como PyPI, Conda-Forge, etc.

> [!IMPORTANT]
> - Si el paquete que va a instalar es de gran tamaño o tarda mucho tiempo en instalarse, afectará al tiempo de inicio de la instancia de Spark.
> - No se admite la modificación de la versión de PySpark, Python, Scala/Java, .NET o Spark.
> - La instalación de paquetes de repositorios externos como PyPI, Conda-Forge o los canales Conda predeterminados no se admiten en áreas de trabajo habilitadas para DEP.

### <a name="install-python-packages"></a>Instalación de paquetes de Python
Los paquetes de Python se pueden instalar desde repositorios como PyPI y Conda-Forge proporcionando un archivo de especificación de entorno. 

#### <a name="environment-specification-formats"></a>Formatos de especificación de entorno

##### <a name="pip-requirementstxt"></a>requirements.txt de PIP
Se puede usar un archivo *requirements.txt* (salida del comando `pip freeze`) para actualizar el entorno. Cuando se actualiza un grupo, los paquetes enumerados en este archivo se descargan desde PyPI. Después, las dependencias completas se almacenan en caché y se guardan para reutilizar el grupo más adelante. 

En el fragmento de código siguiente se muestra el formato del archivo de requisitos. Se enumera el nombre del paquete PyPI, junto con una versión exacta. Este archivo sigue el formato descrito en la documentación de referencia de [pip freeze](https://pip.pypa.io/en/stable/reference/pip_freeze/). 

Este ejemplo fija una versión específica. 
```
absl-py==0.7.0
adal==1.2.1
alabaster==0.7.10
```
##### <a name="yml-format-preview"></a>Formato YML (versión preliminar)
También puede proporcionar un archivo *environment.yml* para actualizar el entorno del grupo. Los paquetes que se enumeran en este archivo se descargan de los canales predeterminados de Conda, de Conda-Forge y de PyPI. Puede especificar otros canales o quitar los canales predeterminados mediante las opciones de configuración.

En este ejemplo se especifican los canales y las dependencias de Conda y PyPI. 

```
name: stats2
channels:
- defaults
dependencies:
- bokeh
- numpy
- pip:
  - matplotlib
  - koalas==1.7.0
```
Para obtener más información, consulte [cómo crear un entorno desde un archivo environment.yml](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#activating-an-environment
).

#### <a name="update-python-packages"></a>Actualización de paquetes de Python
Una vez que haya identificado el archivo de especificación de entorno o el conjunto de bibliotecas que quiere instalar en el grupo de Spark, puede actualizar las bibliotecas de grupos de Spark desde Azure Synapse Studio o Azure Portal. Aquí puede proporcionar la especificación de entorno y seleccionar las bibliotecas de área de trabajo que se van a instalar. 

Después de guardar los cambios, un trabajo de Spark ejecutará la instalación y almacenará en caché el entorno resultante para reutilizarlo más adelante. Una vez completado el trabajo, los nuevos trabajos o sesiones de cuaderno de Spark usarán las bibliotecas de grupos actualizadas. 

##### <a name="manage-packages-from-azure-synapse-studio-or-azure-portal"></a>Administración de paquetes desde Azure Synapse Studio o Azure Portal
Las bibliotecas de grupos de Spark se pueden administrar desde Azure Synapse Studio o Azure Portal. 

Para actualizar o agregar bibliotecas a un grupo de Spark:
1. Navegue hasta el área de trabajo de Azure Synapse Analytics desde Azure Portal.

    Si va a actualizar desde **Azure Portal**:

    - En la sección **Synapse resources** (Recursos de Synapse), seleccione la pestaña **Apache Spark pools** y seleccione un grupo de Spark en la lista.
     
    - Seleccione **Paquetes** en la sección **Configuración** del grupo de Spark.
  
    ![Captura de pantalla que resalta el botón de carga de un archivo de configuración de entorno.](./media/apache-spark-azure-portal-add-libraries/apache-spark-add-library-azure.png "Adición de bibliotecas de Python")
   
    Si va a actualizar desde **Synapse Studio**:
    - Seleccione **Administrar** en el panel de navegación principal y, a continuación, seleccione **Grupos de Apache Spark**.

    - Seleccione la sección **Paquetes** de un grupo de Spark específico.
    ![Captura de pantalla en la que se resalta la opción para cargar un archivo de configuración de entorno en Synapse Studio.](./media/apache-spark-azure-portal-add-libraries/studio-update-libraries.png "Adición de bibliotecas de Python desde Synapse Studio")
   
2. Cargue el archivo de configuración de entorno mediante el selector de archivos en la sección **Paquetes** de la página.
3. También puede seleccionar **paquetes de área de trabajo** adicionales para agregarlos al grupo. 
4. Después de guardar los cambios, se desencadenará un trabajo del sistema para instalar y almacenar en caché las bibliotecas especificadas. Este proceso ayuda a reducir el tiempo general de inicio de sesión. 
5. Una vez que el trabajo se haya completado correctamente, todas las sesiones nuevas seleccionarán las bibliotecas de grupos actualizadas.

> [!IMPORTANT]
> Al seleccionar la opción para **forzar configuración nueva**, finalizarán todas las sesiones actuales del grupo de Spark seleccionado. Una vez finalizadas las sesiones, tendrá que esperar a que el grupo se reinicie. 
>
> Si esta opción no está seleccionada, tendrá que esperar a que la sesión de Spark actual finalice o detenerla manualmente. Una vez finalizada la sesión, deberá dejar que el grupo se reinicie.


##### <a name="track-installation-progress-preview"></a>Seguimiento del progreso de la instalación (versión preliminar)
Cada vez que se actualiza un grupo con un nuevo conjunto de bibliotecas, se inicia un trabajo de Spark reservado para el sistema. Este trabajo de Spark ayuda a supervisar el estado de la instalación de las bibliotecas. Si se produce un error en la instalación debido a conflictos de la biblioteca u otros problemas, el grupo de Spark volverá a su estado anterior o predeterminado. 

Además, los usuarios pueden inspeccionar los registros de instalación para identificar conflictos de dependencia o ver qué bibliotecas se instalaron durante la actualización del grupo.

Para consultar estos registros:
1. Vaya a la lista de aplicaciones de Spark en la pestaña **Supervisar**. 
2. Seleccione el trabajo del sistema de la aplicación de Spark que corresponde a la actualización del grupo. Estos trabajos del sistema se ejecutan con el nombre *SystemReservedJob-LibraryManagement*.
   ![Captura de pantalla que resalta el trabajo de biblioteca reservado para el sistema.](./media/apache-spark-azure-portal-add-libraries/system-reserved-library-job.png "Visualización del trabajo de biblioteca del sistema")
3. Cambie para ver los registros del **controlador** y de **stdout**. 
4. En los resultados, verá los registros relacionados con la instalación de las dependencias.
    ![Captura de pantalla que resalta los resultados del trabajo de biblioteca reservado para el sistema.](./media/apache-spark-azure-portal-add-libraries/system-reserved-library-job-results.png "Visualización del progreso del trabajo de biblioteca del sistema")

## <a name="install-wheel-files"></a>Instalación de archivos wheel
Los archivos wheel de Python son una forma habitual de empaquetar bibliotecas de Python. En Azure Synapse Analytics, los usuarios pueden cargar sus archivos wheel en una ubicación conocida de la cuenta de Azure Data Lake Storage, o bien pueden cargarlos mediante la interfaz de paquetes de área de trabajo de Azure Synapse.

### <a name="workspace-packages-preview"></a>Paquetes de área de trabajo (versión preliminar)
Los paquetes de área de trabajo pueden ser archivos wheel personalizados o privados. Puede cargar estos paquetes en su área de trabajo y después asignarlos a un grupo de Spark específico.

Para agregar paquetes de área de trabajo:
1. Vaya a la pestaña **Administrar** > **Paquetes de área de trabajo**.
2. Cargue sus archivos wheel mediante el selector de archivos.
3. Una vez cargados en el área de trabajo de Azure Synapse, puede agregar estos paquetes a un grupo de Apache Spark específico.

![Captura de pantalla en la que se resaltan los paquetes de área de trabajo.](./media/apache-spark-azure-portal-add-libraries/studio-add-workspace-package.png "Visualización de paquetes de área de trabajo")

>[!WARNING]
>- En Azure Synapse, un grupo de Apache Spark puede aprovechar las bibliotecas personalizadas que se cargan como paquetes de área de trabajo o que se cargan en una ruta de acceso conocida de Azure Data Lake Storage. Sin embargo, estas dos opciones no se pueden usar simultáneamente en el mismo grupo de Apache Spark. Si se proporcionan paquetes usando ambos métodos, solo se instalarán los archivos wheel especificados en la lista de paquetes de área de trabajo. 
>
>- Cuando se usan paquetes de área de trabajo (versión preliminar) para instalar paquetes en un determinado grupo de Apache Spark, existe una limitación que impide seguir especificando paquetes mediante la ruta de acceso de la cuenta de almacenamiento en el mismo grupo.  

### <a name="storage-account"></a>Cuenta de almacenamiento
Los paquetes wheel de compilación personalizada se pueden instalar en el grupo de Apache Spark mediante la carga de todos los archivos wheel en la cuenta de Azure Data Lake Storage (Gen2) que está vinculada al área de trabajo de Synapse. 

Los archivos se deben cargar en la siguiente ruta de acceso en el contenedor predeterminado de la cuenta de almacenamiento: 

```
abfss://<file_system>@<account_name>.dfs.core.windows.net/synapse/workspaces/<workspace_name>/sparkpools/<pool_name>/libraries/python/
```

>[!WARNING]
> En algunos casos, es posible que deba crear la ruta de acceso del archivo según la estructura anterior si aún no existe. Por ejemplo, es posible que tenga que agregar la carpeta ```python``` dentro de la carpeta ```libraries``` si aún no existe.

> [!IMPORTANT]
> Para instalar bibliotecas personalizadas mediante el método de Azure Data Lake Storage, debe tener los permisos **Colaborador de datos de Storage Blob** o **Propietario de datos de Storage Blob** en la cuenta principal de Storage Gen2 que está vinculada con el área de trabajo de Azure Synapse Analytics.


## <a name="session-scoped-packages-preview"></a>Paquetes de ámbito de sesión (versión preliminar)
Además de los paquetes de nivel de grupo, también puede especificar bibliotecas de ámbito de sesión al comienzo de una sesión del cuaderno.  Las bibliotecas de ámbito de sesión permiten especificar y usar entornos de Python personalizados en una sesión de cuaderno. 

Cuando se usan bibliotecas de ámbito de sesión, es importante tener en cuenta lo siguiente:
   - Al instalar las bibliotecas de ámbito de sesión, solo el cuaderno actual tiene acceso a las bibliotecas especificadas. 
   - Estas bibliotecas no afectarán a otras sesiones o trabajos que usen el mismo grupo de Spark. 
   - Las bibliotecas se instalan sobre las bibliotecas del entorno de ejecución base y de nivel de grupo. 
   - Las bibliotecas de cuadernos tienen la prioridad más alta.

Para especificar los paquetes de ámbito de sesión:
1.  Vaya al grupo de Spark seleccionado y asegúrese de que ha habilitado las bibliotecas de nivel de sesión.  Para habilitar esta configuración, vaya a la pestaña **Administrar** > **Grupo de Apache Spark** > **Paquetes**. ![Habilitación de paquetes de sesión.](./media/apache-spark-azure-portal-add-libraries/enable-session-packages.png "Habilitación de paquetes de sesión")
2.  Una vez aplicada la configuración, puede abrir un cuaderno y seleccionar **Configure Session** (Configurar sesión) > **Paquetes**.
  ![Especificación de paquetes de sesión.](./media/apache-spark-azure-portal-add-libraries/update-session-notebook.png "Actualización de la configuración de sesión")
3.  Aquí puede cargar un archivo *environment.yml* de Conda para instalar o actualizar paquetes dentro de una sesión. Una vez iniciada la sesión, se instalarán las bibliotecas especificadas. Cuando finalice la sesión, estas bibliotecas ya no estarán disponibles, ya que son específicas de la sesión.

## <a name="verify-installed-libraries"></a>Comprobación de las bibliotecas instaladas
Para comprobar si se han instalado las versiones correctas de las bibliotecas adecuadas desde PyPI, ejecute el código siguiente:
```python
import pkg_resources
for d in pkg_resources.working_set:
     print(d)
```
En algunos casos, para ver las versiones de los paquetes desde Conda, es posible que deba inspeccionar la versión de cada paquete individualmente.

## <a name="next-steps"></a>Pasos siguientes
- Visualización de las bibliotecas predeterminadas: [Compatibilidad de las versiones de Azure Spark](apache-spark-version-support.md)
- [Solución de errores de instalación de biblioteca](apache-spark-troubleshoot-library-errors.md)
- Creación de un canal privado de Conda mediante la cuenta de Azure Data Lake Storage: [canales privados de Conda](./spark/../apache-spark-custom-conda-channel.md)
