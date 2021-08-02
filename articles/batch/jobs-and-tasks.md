---
title: Trabajos y tareas en Azure Batch
description: Obtenga información sobre trabajos y tareas y cómo se usan en un flujo de trabajo de Azure Batch desde el punto de vista del desarrollo.
ms.topic: conceptual
ms.date: 06/11/2021
ms.openlocfilehash: faedb912b0c21acdec977fe7651f0a5daddb2c6f
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112004176"
---
# <a name="jobs-and-tasks-in-azure-batch"></a>Trabajos y tareas en Azure Batch

En Azure Batch, una *tarea* representa una unidad de cálculo. Un *trabajo* es una colección de estas tareas. A continuación se describen los trabajos y las tareas y cómo se usan en un flujo de trabajo de Azure Batch.

## <a name="jobs"></a>Trabajos

Un trabajo es una colección de tareas. El trabajo administra cómo sus tareas realizan el cálculo en los nodos de proceso de un grupo.

Un trabajo especifica el [grupo](nodes-and-pools.md#pools) en el que se va a ejecutar la labor. Puede crear un grupo para cada trabajo o usar uno solo para muchos de ellos. Puede crear un grupo para cada trabajo asociado con una [programación de trabajo](#scheduled-jobs) o un grupo para todos los trabajos asociados con esta.

### <a name="job-priority"></a>prioridad del trabajo

Puede asignar una prioridad de trabajo opcional a los trabajos que cree. El servicio Batch usa el valor de prioridad del trabajo para determinar el orden de programación (para todas las tareas dentro del trabajo) en cada grupo.

Para actualizar la prioridad de un trabajo, llame a la operación [Actualizar las propiedades de un trabajo](/rest/api/batchservice/job/update) (REST de Batch) o modifique [CloudJob.Priority](/dotnet/api/microsoft.azure.batch.cloudjob.priority) (Batch para .NET). Los valores de prioridad van de -1000 (prioridad mínima) a 1000 (prioridad máxima).

Dentro del mismo grupo, los trabajos de mayor prioridad tienen precedencia de programación sobre los trabajos con menor prioridad. Las tareas en trabajos de prioridad más baja que ya se están ejecutando no se reemplazarán por las tareas de un trabajo de mayor prioridad. Los trabajos con el mismo nivel de prioridad tienen la misma probabilidad de programarse, y el orden de ejecución de las tareas no está definido.

Un trabajo con un valor de prioridad alto que se ejecute en un grupo no afectará a la programación de trabajos que se ejecutan en un grupo independiente o en una cuenta de Batch distinta. La prioridad del trabajo no se aplica a los [grupos automáticos](nodes-and-pools.md#autopools), que se crean cuando se envía el trabajo.

### <a name="job-constraints"></a>Restricciones del trabajo

Puede utilizar restricciones de trabajo para especificar determinados límites para sus trabajos:

- Puede establecer un **tiempo de reloj máximo**, de modo que si un trabajo lleva más tiempo que el especificado, el trabajo y todas sus tareas se finalizarán.
- Puede especificar el **número máximo de reintentos de tarea** como restricción. Puede incluso especificar si una tarea se debe reintentar siempre o no se debe reintentar nunca. Volver a intentar una tarea significa que hay un error en ella, se vuelve a poner en cola para ejecutarse de nuevo.

### <a name="job-manager-tasks-and-automatic-termination"></a>Tareas del administrador de trabajos y terminación automática

La aplicación cliente puede agregar tareas a un trabajo o puede especificar una [tarea del administrador de trabajos](#job-manager-task). Una tarea del administrador de trabajos contiene la información necesaria para crear las tareas que se requieren para un trabajo. Estas tareas se ejecutan en uno de los nodos de proceso del grupo. El servicio Batch controla específicamente las tareas del administrador de trabajos; las pone en cola en cuanto se crea el trabajo y las reinicia si se produce un error. Se requiere una tarea del administrador de trabajos para los trabajos creados mediante una [programación de trabajo](#scheduled-jobs), ya que es la única manera de definir las tareas antes de que se cree una instancia del trabajo.

De manera predeterminada, los trabajos permanecen en estado activo cuando se completan todas las tareas que contienen. Este comportamiento se puede cambiar, con el fin de que el trabajo finalice automáticamente cuando se completen todas sus tareas. Establezca la propiedad **onAllTasksComplete** ([OnAllTasksComplete](/dotnet/api/microsoft.azure.batch.cloudjob) en .NET de Batch) en `terminatejob`*` para que el trabajo finalice automáticamente cuando todas sus tareas estén en estado completado.

El servicio Batch considera que un trabajo *sin* tareas tiene todas sus tareas completadas. Por lo tanto, esta opción se utiliza normalmente con una [tarea del administrador de trabajos](#job-manager-task). Si quiere usar la finalización automática de trabajos sin un administrador de trabajos, debe establecer inicialmente la propiedad **onAllTasksComplete** de cada trabajo nuevo en `noaction` y, después, establecerla en `terminatejob`, pero solo después de que haya terminado de agregar tareas al trabajo.

### <a name="scheduled-jobs"></a>Scheduled jobs

Las [programaciones de trabajos](/rest/api/batchservice/jobschedule) permiten crear trabajos recurrentes dentro del servicio Batch. Una programación de trabajo especifica cuándo se deben ejecutar los trabajos e incluye las especificaciones de los trabajos que se van a ejecutar. Puede especificar la duración de la programación (cuánto dura y cuándo está vigente), y con qué frecuencia se crean trabajos durante el período programado.

## <a name="tasks"></a>Tareas

Una tarea es una unidad de cálculo que está asociada con un trabajo. Se ejecuta en un nodo. Las tareas se asignan a un nodo para su ejecución o se ponen en cola hasta que quede libre un nodo. Es decir, una tarea ejecuta uno o más programas o scripts en un nodo de proceso para llevar a cabo el trabajo necesario.

Cuando crea una tarea, puede especificar:

- La **línea de comandos** de la tarea. Se trata de la línea de comandos que ejecuta la aplicación o el script en el nodo de proceso.

    Es importante comprobar que la línea de comandos no se ejecute en un shell. Por tanto, no se podrá beneficiar de forma nativa de las características de shell como la expansión de [variables de entorno](#environment-settings-for-tasks) (esto incluye `PATH`). Para beneficiarse de estas características, debe invocar el shell en la línea de comandos, como puede ser iniciando `cmd.exe` en nodos de Windows o `/bin/sh` en Linux:

    `cmd /c MyTaskApplication.exe %MY_ENV_VAR%`

    `/bin/sh -c MyTaskApplication $MY_ENV_VAR`

    Si las tareas deben ejecutar una aplicación o un script que no se encuentra en `PATH` en el nodo o en las variables de entorno de referencia, invoque el shell explícitamente en la línea de comandos de la tarea.
- **archivos de recursos** que contienen los datos que se procesan. Estos archivos se copian automáticamente en el nodo desde Blob Storage en una cuenta de Azure Storage antes de que se ejecute la línea de comandos de la tarea. Para más información, consulte [Tarea de inicio](#start-task) y [Archivos y directorios](files-and-directories.md).
- Las **variables de entorno** que requiere la aplicación. Para más información, consulte [Configuración del entorno para las tareas](#environment-settings-for-tasks).
- Las **restricciones** con que se debe ejecutar la tarea. Por ejemplo, las restricciones incluyen el tiempo máximo que se permite que la tarea se ejecute, el número máximo de veces que se debe volver a intentar una tarea si se produce un error y el tiempo máximo que se conservan los archivos en el directorio de trabajo de la tarea.
- **Application packages** que se implementan en el nodo de proceso en el que está programado que se ejecute la tarea. [Application packages](batch-application-packages.md) proporcionan una implementación simplificada y el control de las versiones de las aplicaciones que ejecutan las tareas. Los paquetes de aplicaciones de nivel de tarea son especialmente útiles en entornos de grupo compartido, donde los distintos trabajos se ejecutan en un grupo que no se elimina cuando se completa un trabajo. Si el trabajo tiene menos tareas que nodos en el grupo, los paquetes de aplicación de las tareas pueden minimizar la transferencia de datos, ya que la aplicación se implementa solo en los nodos que ejecutan tareas.
- Una referencia de **imagen de contenedor** de Docker Hub o un registro privado y valores de configuración adicionales para crear un contenedor de Docker en el que la tarea se ejecute en el nodo. Esta información solo se especifica si el grupo está configurado con una configuración de contenedor.

> [!NOTE]
> La duración máxima de una tarea, desde el momento en que se agrega al trabajo hasta que se completa, es de 180 días. Las tareas completadas se mantienen durante siete días; los datos de las tareas no completadas dentro de la duración máxima no están accesibles.

Además de las tareas que se definen para realizar cálculos en un nodo, el servicio Batch también proporciona varias tareas especiales:

- [Tarea de inicio](#start-task)
- [Tareas del Administrador de trabajos](#job-manager-task)
- [Tareas de preparación y liberación de trabajos](#job-preparation-and-release-tasks)
- [Tareas de instancias múltiples](#multi-instance-task)
- [Dependencias de las tareas](#task-dependencies)

### <a name="start-task"></a>Tarea de inicio

Al asociar una tarea de inicio con un grupo, puede preparar el entorno operativo de sus nodos. Por ejemplo, puede realizar acciones como instalar las aplicaciones que las tareas ejecutan o iniciar procesos en segundo plano. La tarea de inicio se ejecuta cada vez que se inicia un nodo, siempre y cuando permanezca en el grupo. Esto incluye cuando el nodo se agrega por primera vez al grupo y cuando se reinicia o se restablece la imagen inicial.

Una de las principales ventajas de la tarea de inicio es que puede contener toda la información necesaria para configurar un nodo de proceso e instalar las aplicaciones necesarias para ejecutar tareas. Por lo tanto, aumentar el número de nodos en un grupo es tan simple como especificar el nuevo número de nodos de destino. La tarea de inicio proporciona la información necesaria para el servicio Batch para configurar los nuevos nodos y prepararlos para aceptar tareas.

Como con cualquier tarea de Azure Batch, se puede especificar una lista de archivos de recursos en [Azure Storage](../storage/index.yml), además de una línea de comandos para ejecutar. El servicio Batch copia primero los archivos de recursos desde Azure Storage al nodo y después ejecuta la línea de comandos. Para una tarea de inicio de grupo, la lista de archivos contiene normalmente la aplicación de la tarea y sus dependencias.

Sin embargo, la tarea de inicio también puede incluir datos de referencia que se utilizarán en todas las tareas que se ejecuten en el nodo de proceso. Por ejemplo, la línea de comandos de una tarea de inicio podría realizar una operación `robocopy` para copiar los archivos de la aplicación (que se especificaron como archivos de recursos y se descargaron en el nodo) del [directorio de trabajo](files-and-directories.md) de la tarea de inicio a la **carpeta compartida** y después ejecutar un archivo MSI o `setup.exe`.

Normalmente, querrá que el servicio Batch espere a que se complete la tarea de inicio antes de considerar que el nodo está listo para asignarse a tareas. Sin embargo, puede configurarlo de forma diferente según sea necesario.

Si una tarea de inicio da error en un nodo de proceso, el estado del nodo se actualiza para reflejar el error y no se le asigna al nodo ninguna tarea. Una tarea de inicio puede generar un error si existe un problema al copiar sus archivos de recursos desde el almacenamiento o si el proceso ejecutado mediante su línea de comandos devuelve un código de salida distinto de cero.

Si agrega una tarea de inicio a un grupo existente o la actualiza, debe reiniciar sus nodos de proceso para que la tarea de inicio se les aplique.

>[!NOTE]
> Batch limita el tamaño total de una tarea de inicio, que incluye archivos de recursos y variables de entorno. Si necesita reducir el tamaño de una tarea de inicio, puede usar uno de estos dos enfoques:
>
> 1. Puede utilizar paquetes de aplicación para distribuir las aplicaciones o los datos por los nodos del grupo de Batch. Para más información sobre los paquetes de aplicación, consulte [Implementación de aplicaciones en nodos de proceso con paquetes de aplicaciones de Batch](batch-application-packages.md).
> 2. Puede crear manualmente un archivo comprimido que contenga los archivos de aplicación. Cargue el archivo comprimido en Azure Storage como blob. Defina el archivo comprimido como archivo de recursos para la tarea de inicio. Antes de ejecutar la línea de comandos para la tarea de inicio, descomprima el archivo desde la línea de comandos. 
>
>    Para ello, puede usar la herramienta de archivado que prefiera. Debe incluir la herramienta que utilice para descomprimir el archivo como archivo de recursos para la tarea de inicio.

### <a name="job-manager-task"></a>Tareas del Administrador de trabajos

Normalmente se utiliza una tarea del administrador de trabajos para controlar y/o supervisar la ejecución del trabajo. Por ejemplo, las tareas del administrador de trabajos suelen usarse para crear y enviar las tareas de un trabajo, determinar las tareas adicionales que se deben ejecutar y determinar si el trabajo ha finalizado.

Sin embargo, una tarea del administrador de trabajos no se limita a estas actividades. Es una tarea completa que puede realizar todas las acciones que se requieren para el trabajo. Por ejemplo, una tarea del administrador de trabajos podría descargar un archivo especificado como un parámetro, analizar el contenido de ese archivo y enviar tareas adicionales en función de ese contenido.

Una tarea del administrador de trabajos se inicia antes que cualquier otra. Ofrece las siguientes características:

- Se envía automáticamente como una tarea mediante el servicio Batch cuando se crea el trabajo.
- Está programada para ejecutarse antes que las otras tareas de un trabajo.
- Su nodo asociado es el último en eliminarse de un grupo cuando se reduce el tamaño del grupo.
- Su finalización se puede vincular a la finalización de todas las tareas en el trabajo.
- A una tarea del administrador de trabajos se le otorga la más alta prioridad cuando es necesario reiniciarla. Si un nodo inactivo no está disponible, el servicio Batch puede finalizar una de las otras tareas en ejecución del grupo para dejar espacio para la ejecución de la tarea del administrador de trabajos.
- Una tarea del Administrador de trabajos de un trabajo no tiene prioridad sobre las tareas de otros trabajos. Entre trabajos, solo se tienen en cuenta las prioridades de nivel de trabajo.

### <a name="job-preparation-and-release-tasks"></a>Tareas de preparación y liberación de trabajos

Batch proporciona tareas de preparación de trabajos para la configuración de la ejecución previa al trabajo y tareas de liberación de trabajos para el mantenimiento o la limpieza posteriores al trabajo.

una tarea de preparación del trabajo se ejecuta en todos los nodos de proceso programados para ejecutar tareas, antes de que se ejecute ninguna otra tarea del trabajo. Por ejemplo, puede usar una tarea de preparación del trabajo para copiar los datos compartidos por todas las tareas, pero que sean únicos para el trabajo.

cuando un trabajo se ha completado, la tarea de liberación del trabajo se ejecuta en cada nodo del grupo que ejecutó al menos una tarea. Por ejemplo, una tarea de liberación del trabajo puede eliminar datos que la tarea de preparación del trabajo copió, o comprimir y cargar los datos del registro de diagnóstico.

Tanto para la tarea de preparación del trabajo como para las de liberación, puede especificar una línea de comandos que se ejecute cuando se invoque la tarea. Las tareas ofrecen características tales como descarga de archivos, ejecución con elevación de privilegios, variables de entorno personalizadas, duración máxima de ejecución, número de reintentos y tiempo de retención de archivo.

Para obtener más información sobre las tareas de preparación y liberación del trabajo, consulte [Ejecución de las tareas de preparación y realización de trabajos en los nodos de ejecución de Azure Batch](batch-job-prep-release.md).

### <a name="multi-instance-task"></a>Tarea de instancias múltiples

Una [tarea de instancias múltiples](batch-mpi.md) es aquella que está configurada para ejecutarse simultáneamente en varios nodos de proceso. Con tareas de instancias múltiples, puede habilitar escenarios de informática de alto rendimiento que requieren tener un grupo de nodos de proceso asignados juntos para procesar una carga de trabajo única, como la interfaz de paso de mensajes (MPI).

Para ver una explicación detallada sobre la ejecución de trabajos de MPI en el servicio Batch mediante la biblioteca .NET de Batch, consulte [Uso de tareas de instancias múltiples para ejecutar aplicaciones de la Interfaz de paso de mensajes (MPI) en Azure Batch](batch-mpi.md).

### <a name="task-dependencies"></a>Dependencias de las tareas

Las [dependencias de tareas](batch-task-dependencies.md), como el propio nombre implica, permiten especificar que una tarea depende de que se completen otras tareas antes de su ejecución. Esta característica proporciona compatibilidad con aquellas situaciones en las que una tarea "de bajada" consume el resultado de una tarea "del canal de subida", o cuando una tarea del canal de subida realiza alguna inicialización requerida por una tarea de bajada.

Para utilizar esta característica, primero debe [habilitar las dependencias de tareas](batch-task-dependencies.md#enable-task-dependencies
) en su trabajo de Batch. Luego, en cada tarea que dependa de otra (o de muchas otras), especifique las tareas de las que depende dicha tarea.

Con las dependencias de tareas, se pueden configurar escenarios como los siguientes:

- *taskB* depende de *taskA* (la ejecución de *taskB* no se iniciará hasta que *taskA* se haya completado).
- *taskC* depende de *taskA* y *taskB*.
- *taskD* depende de un intervalo de tareas, como las tareas de *1* a *10*, antes de ejecutarse.

Para más detalles, consulte [Dependencias de tareas en Azure Batch](batch-task-dependencies.md) y el código de ejemplo de [TaskDependencies](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp/ArticleProjects/TaskDependencies) en el repositorio de ejemplos de GitHub [azure-batch-samples](https://github.com/Azure-Samples/azure-batch-samples).

### <a name="environment-settings-for-tasks"></a>Configuración del entorno para las tareas

Cada tarea que ejecuta el servicio Batch tiene acceso a las variables de entorno que establece en los nodos de proceso. Esto incluye las [variables de entorno definidas por el servicio Batch](./batch-compute-node-environment-variables.md) y las variables de entorno personalizadas que se pueden definir para las tareas. Las aplicaciones y los scripts que ejecutan las tareas tienen acceso a estas variables de entorno durante la ejecución.

Puede establecer variables de entorno personalizadas en el nivel de tarea o de trabajo rellenando la propiedad *environment settings* de estas entidades. Para obtener más detalles, consulte la operación [Incorporación de una tarea a un trabajo](/rest/api/batchservice/task/add?) (REST de Batch) o las propiedades [CloudTask.EnvironmentSettings](/dotnet/api/microsoft.azure.batch.cloudtask.environmentsettings) y [CloudJob.CommonEnvironmentSettings](/dotnet/api/microsoft.azure.batch.cloudjob.commonenvironmentsettings) en Batch para .NET.

La aplicación cliente o el servicio pueden obtener las variables de entorno de una tarea, tanto las definidas por el servicio como las personalizadas, mediante la operación [Get information about a task](/rest/api/batchservice/task/get) (Obtener información acerca de una tarea) (REST de Batch) o accediendo a la propiedad [CloudTask.EnvironmentSettings](/dotnet/api/microsoft.azure.batch.cloudtask.environmentsettings) (Batch para .NET). Los procesos que se ejecutan en un nodo de proceso pueden acceder a estas y a otras variables de entorno en el nodo, por ejemplo mediante la conocida sintaxis de `%VARIABLE_NAME%` (Windows) o `$VARIABLE_NAME` (Linux).

Puede encontrar una lista de las variables de entorno definidas por el servicio en [Compute node environment variables](batch-compute-node-environment-variables.md) (Variables de entorno en los nodos de proceso).

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre [archivos y directorios](files-and-directories.md).
