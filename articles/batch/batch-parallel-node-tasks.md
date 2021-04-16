---
title: Ejecución simultánea de tareas para maximizar el uso de los nodos de proceso de Batch
description: Aumente la eficacia y reduzca los costos usando menos nodos de ejecución y ejecutando tareas en paralelo en cada nodo de un grupo de Azure Batch
ms.topic: how-to
ms.date: 03/25/2021
ms.custom: H1Hack27Feb2017, devx-track-csharp
ms.openlocfilehash: 2a8f2d6a040bee0e32359f4860d7b346ac08c48e
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105607990"
---
# <a name="run-tasks-concurrently-to-maximize-usage-of-batch-compute-nodes"></a>Ejecución simultánea de tareas para maximizar el uso de los nodos de proceso de Batch

Mediante la ejecución simultánea de más de una tarea en cada nodo del grupo de Azure Batch, puede maximizar el uso de los recursos en un menor número de nodos de ejecución en el grupo.

Aunque algunos escenarios funcionan mejor con todos los recursos de un nodo dedicados a una sola tarea, en algunas cargas de trabajo se pueden observar tiempos de trabajo más cortos y menores costos si varias tareas comparten esos recursos. Considere los siguientes escenarios:

- **Reducción al mínimo de la transferencia de datos** de las tareas que pueden compartir datos. Puede reducir considerablemente los gastos de transferencia de datos copiando los datos compartidos en un número menor de nodos y ejecutando tareas en paralelo en cada nodo. Esto es válido especialmente si los datos que se copian en cada nodo deben transferirse entre regiones geográficas.
- **Maximización del uso de memoria** para las tareas que requieren una gran cantidad de memoria, pero solo durante períodos breves y en momentos variables durante la ejecución. Puede emplear menos nodos de ejecución, pero de mayor tamaño y con más memoria, para controlar de forma eficaz dichos aumentos. Estos nodos tendrán varias tareas ejecutándose en paralelo en cada nodo, pero cada tarea puede aprovechar la abundante memoria de los nodos en distintos momentos.
- **Mitigación de los límites de número de nodos** cuando se requiere la comunicación entre nodos dentro de un grupo. Actualmente, los grupos configurados para la comunicación entre nodos están limitados a 50 nodos de ejecución. Si cada uno de los nodos de este tipo de grupo es capaz de ejecutar tareas en paralelo, el número de tareas que se podrán ejecutar simultáneamente será mayor.
- **Replicación en un clúster de proceso local**, como cuando mueve por primera vez un entorno de procesos a Azure. Si la solución local existente ejecuta varias tareas en cada nodo de proceso, puede aumentar el número máximo de tareas de nodo para que refleje con mayor fidelidad esa configuración.

## <a name="example-scenario"></a>Escenario de ejemplo

Por ejemplo, imagine una aplicación de tarea con unos requisitos de CPU y memoria tales que unos nodos [Estándar\_D1](../cloud-services/cloud-services-sizes-specs.md#d-series) sean suficientes. Sin embargo, para ejecutar el trabajo en el tiempo requerido, se necesitan 1000 nodos de ese tipo.

En lugar de utilizar nodos Standard\_D1 con un núcleo de CPU, podría utilizar nodos [Standard\_D14](../cloud-services/cloud-services-sizes-specs.md#d-series) con 16 núcleos en cada nodo y habilitar la ejecución de tareas en paralelo. De este modo, se podría usar un número de nodos 16 veces menor; es decir, en lugar de 1000 nodos, solo serían necesarios 63. Si se necesitan datos de referencia o archivos de aplicación de gran tamaño para cada nodo, la eficiencia y la duración del trabajo se mejoran, ya que los datos se copian en solo 63 nodos.

## <a name="enable-parallel-task-execution"></a>Habilitación de la ejecución en paralelo de tareas

Los nodos de proceso para la ejecución en paralelo de tareas se configuran a nivel de grupo. Con la biblioteca .NET de Batch, establezca la propiedad [CloudPool.TaskSlotsPerNode](/dotnet/api/microsoft.azure.batch.cloudpool.taskslotspernode) al crear un grupo. Si usa la API REST de Batch, establezca el elemento [taskSlotsPerNode](/rest/api/batchservice/pool/add) en el cuerpo de la solicitud durante la creación del grupo.

> [!NOTE]
> Solo puede establecer el elemento `taskSlotsPerNode` y la propiedad [TaskSlotsPerNode](/dotnet/api/microsoft.azure.batch.cloudpool) en el momento de crear el grupo. No se pueden modificar después de haber creado el grupo.

Azure Batch permite una configuración de ranuras de tarea por nodo que cuadriplica el número de núcleos de nodo. Por ejemplo, si el grupo está configurado con nodos de tamaño "Grande" (cuatro núcleos), `taskSlotsPerNode` se puede establecer en 16. Sin embargo, independientemente de cuántos núcleos tiene el nodo, no puede tener más de 256 ranuras de tarea por nodo. Para más información sobre el número de núcleos de cada uno de los tamaños de nodo, consulte [Tamaños de Cloud Services](../cloud-services/cloud-services-sizes-specs.md). Para más información sobre los límites del servicio, consulte [Cuotas y límites del servicio Azure Batch](batch-quota-limit.md).

> [!TIP]
> Asegúrese de tener en cuenta el valor `taskSlotsPerNode` al construir una [fórmula de escalado automático](/rest/api/batchservice/pool/enableautoscale) para el grupo. Por ejemplo, una fórmula que evalúe `$RunningTasks` podría verse afectada considerablemente por un aumento en las tareas por nodo. Para más información, consulte [Escalación automática de los nodos de ejecución en un grupo de Azure Batch](batch-automatic-scaling.md).

## <a name="specify-task-distribution"></a>Especificación de la distribución de tareas

A la hora de habilitar tareas simultáneas, es importante especificar cómo desea que se distribuyan las tareas entre los nodos del grupo.

Mediante la propiedad [CloudPool.TaskSchedulingPolicy](/dotnet/api/microsoft.azure.batch.cloudpool.taskschedulingpolicy), puede especificar que las tareas se deberían asignar de manera uniforme entre todos los nodos del grupo ("propagación"). O bien, puede especificar que se deberían asignar todas las tareas posibles a cada nodo antes de asignarlas a otro nodo del grupo ("empaquetado").

Como ejemplo, considere el grupo de nodos [Estándar\_D14](../cloud-services/cloud-services-sizes-specs.md#d-series) (en el ejemplo anterior) configurado con un valor [CloudPool.TaskSlotsPerNode](/dotnet/api/microsoft.azure.batch.cloudpool.taskslotspernode) de 16. Si [CloudPool.TaskSchedulingPolicy](/dotnet/api/microsoft.azure.batch.cloudpool.taskschedulingpolicy) se configura con un [ComputeNodeFillType](/dotnet/api/microsoft.azure.batch.common.computenodefilltype) de *Pack*, se podría maximizar el uso de los 16 núcleos de cada nodo y permitir que un [grupo con escalabilidad automática](batch-automatic-scaling.md) elimine los nodos sin usar del grupo (nodos sin tareas asignadas). Esto minimiza el uso de recursos y permite ahorrar dinero.

## <a name="define-variable-slots-per-task"></a>Definición de ranuras variables por tarea

Una tarea se puede definir con la propiedad [CloudTask.RequiredSlots](/dotnet/api/microsoft.azure.batch.cloudtask.requiredslots) especificando el número de ranuras que requiere para ejecutarse en un nodo de ejecución. El valor predeterminado es 1. Puede establecer ranuras de tareas variables si las tareas tienen pesos diferentes en cuanto al uso de recursos en el nodo de ejecución. Esto permite que cada nodo de ejecución tenga un número razonable de tareas en ejecución simultáneas que no llegue a saturar los recursos del sistema, como la CPU o la memoria.

Por ejemplo, en el caso de un grupo con la propiedad `taskSlotsPerNode = 8`, puede enviar tareas de uso intensivo de CPU que requieren varios núcleos con `requiredSlots = 8`, y otras tareas con `requiredSlots = 1`. Cuando esta carga de trabajo mixta está programada, las tareas de uso intensivo de CPU se ejecutarán exclusivamente en sus nodos de ejecución, mientras que otras tareas se podrán ejecutar simultáneamente (hasta ocho tareas a la vez) en otros nodos. Esto le ayuda a equilibrar la carga de trabajo entre los nodos de ejecución y mejorar la eficacia del uso de los recursos.

Asegúrese de que no especifica un valor de `requiredSlots` para la tarea que sea mayor que el valor de `taskSlotsPerNode` para el grupo. Esto hará que la tarea no se pueda ejecutar nunca. El servicio Batch no valida actualmente este conflicto cuando envía tareas, porque es posible que el trabajo no tenga un grupo enlazado en el momento del envío, o que se cambie a un grupo diferente si se deshabilita y se vuelve a habilitar.

> [!TIP]
> Cuando se usan ranuras de tareas variables, es posible que las tareas grandes con más ranuras necesarias no se puedan programar temporalmente debido a que no hay suficientes ranuras disponibles en ningún nodo de ejecución, incluso cuando todavía hay ranuras inactivas en algunos nodos. Puede subir la prioridad del trabajo para estas tareas con el fin de aumentar su oportunidad de competir por las ranuras disponibles en los nodos.
>
> El servicio Batch emite [TaskScheduleFailEvent](batch-task-schedule-fail-event.md) cuando no puede programar una tarea para que se ejecute, y sigue reintentando la programación hasta que están disponibles las ranuras necesarias. Puede escuchar ese evento para detectar posibles problemas de programación de tareas y llevar a cabo su mitigación en consecuencia.

## <a name="batch-net-example"></a>Ejemplo de Batch .NET

En los siguientes fragmentos de código de la API de [.NET para Batch](/dotnet/api/microsoft.azure.batch) se muestra cómo crear un grupo con varias ranuras de tareas por nodo y cómo enviar una tarea con las ranuras necesarias.

### <a name="create-a-pool-with-multiple-task-slots-per-node"></a>Creación de un grupo con varias ranuras de tareas por nodo

En este fragmento de código se muestra una solicitud para crear un grupo que contiene cuatro nodos con cuatro ranuras de tarea por nodo. Se especifica una directiva de programación de tareas que llenará cada nodo de tareas antes de asignarlas a otro nodo del grupo.

Para obtener más información sobre cómo agregar grupos mediante la API de .NET para Batch, consulte [BatchClient.PoolOperations.CreatePool](/dotnet/api/microsoft.azure.batch.pooloperations.createpool).

```csharp
CloudPool pool =
    batchClient.PoolOperations.CreatePool(
        poolId: "mypool",
        targetDedicatedComputeNodes: 4
        virtualMachineSize: "standard_d1_v2",
        cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));

pool.TaskSlotsPerNode = 4;
pool.TaskSchedulingPolicy = new TaskSchedulingPolicy(ComputeNodeFillType.Pack);
pool.Commit();
```

### <a name="create-a-task-with-required-slots"></a>Creación de una tarea con las ranuras necesarias

Este fragmento de código crea una tarea con `requiredSlots` no predeterminado. Esta tarea solo se ejecutará cuando haya suficientes ranuras libres disponibles en un nodo de ejecución.

```csharp
CloudTask task = new CloudTask(taskId, taskCommandLine)
{
    RequiredSlots = 2
};
```

### <a name="list-compute-nodes-with-counts-for-running-tasks-and-slots"></a>Enumeración de los nodos de ejecución con recuentos para ejecutar tareas y ranuras

En este fragmento de código se enumeran todos los nodos de ejecución del grupo y se imprimen los recuentos para ejecutar tareas y ranuras de tarea por nodo.

```csharp
ODATADetailLevel nodeDetail = new ODATADetailLevel(selectClause: "id,runningTasksCount,runningTaskSlotsCount");
IPagedEnumerable<ComputeNode> nodes = batchClient.PoolOperations.ListComputeNodes(poolId, nodeDetail);

await nodes.ForEachAsync(node =>
{
    Console.WriteLine(node.Id + " :");
    Console.WriteLine($"RunningTasks = {node.RunningTasksCount}, RunningTaskSlots = {node.RunningTaskSlotsCount}");

}).ConfigureAwait(continueOnCapturedContext: false);
```

### <a name="list-task-counts-for-the-job"></a>Enumeración de recuentos de tarea para el trabajo

Este fragmento de código obtiene los recuentos de tareas para el trabajo, lo que incluye los recuentos de tareas y de ranuras de tarea por estado de tarea.

```csharp
TaskCountsResult result = await batchClient.JobOperations.GetJobTaskCountsAsync(jobId);

Console.WriteLine("\t\tActive\tRunning\tCompleted");
Console.WriteLine($"TaskCounts:\t{result.TaskCounts.Active}\t{result.TaskCounts.Running}\t{result.TaskCounts.Completed}");
Console.WriteLine($"TaskSlotCounts:\t{result.TaskSlotCounts.Active}\t{result.TaskSlotCounts.Running}\t{result.TaskSlotCounts.Completed}");
```

## <a name="batch-rest-example"></a>Ejemplo de REST Batch

En los siguientes fragmentos de código de la API [REST para Batch](/rest/api/batchservice/) se muestra cómo crear un grupo con varias ranuras de tareas por nodo y cómo enviar una tarea con las ranuras necesarias.

### <a name="create-a-pool-with-multiple-task-slots-per-node"></a>Creación de un grupo con varias ranuras de tareas por nodo

En este fragmento de código, se muestra una solicitud para crear un grupo que contiene dos nodos de gran tamaño con un máximo de cuatro tareas por nodo.

Para más información sobre cómo agregar grupos mediante la API REST, consulte [Agregar un grupo a una cuenta](/rest/api/batchservice/pool/add).

```json
{
  "odata.metadata":"https://myaccount.myregion.batch.azure.com/$metadata#pools/@Element",
  "id":"mypool",
  "vmSize":"large",
  "cloudServiceConfiguration": {
    "osFamily":"4",
    "targetOSVersion":"*",
  },
  "targetDedicatedComputeNodes":2,
  "taskSlotsPerNode":4,
  "enableInterNodeCommunication":true,
}
```

### <a name="create-a-task-with-required-slots"></a>Creación de una tarea con las ranuras necesarias

En este fragmento de código se muestra una solicitud para agregar una tarea con `requiredSlots` no predeterminado. Esta tarea solo se ejecutará cuando haya suficientes ranuras libres disponibles en el nodo de ejecución.

```json
{
  "id": "taskId",
  "commandLine": "bash -c 'echo hello'",
  "userIdentity": {
    "autoUser": {
      "scope": "task",
      "elevationLevel": "nonadmin"
    }
  },
  "requiredSLots": 2
}
```

## <a name="code-sample-on-github"></a>Código de ejemplo en GitHub

El proyecto [ParallelTasks](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ParallelTasks) en GitHub muestra el uso de la propiedad [CloudPool.TaskSlotsPerNode](/dotnet/api/microsoft.azure.batch.cloudpool.taskslotspernode).

Esta aplicación de consola de C# utiliza la biblioteca de [.NET de Batch](/dotnet/api/microsoft.azure.batch) para crear un grupo con uno o más nodos de proceso. Ejecuta un número configurable de tareas en esos nodos para simular una carga variable. La salida de la aplicación especifica qué nodos han ejecutado cada tarea. La aplicación también proporciona un resumen de los parámetros de trabajo y la duración.

A modo de ejemplo, se muestra a continuación la parte de resumen de la salida de dos ejecuciones diferentes de la aplicación de ejemplo ParallelTasks. Las duraciones del trabajo que se muestran aquí no incluyen la hora de creación del grupo, ya que cada trabajo se envió a un grupo creado anteriormente cuyos nodos de ejecución tenían el estado *Inactivo* en el momento del envío.

La primera ejecución de la aplicación de ejemplo muestra que, con un solo nodo en el grupo y la configuración predeterminada de una tarea por nodo, la duración del trabajo es superior a 30 minutos.

```
Nodes: 1
Node size: large
Task slots per node: 1
Max slots per task: 1
Tasks: 32
Duration: 00:30:01.4638023
```

La segunda ejecución del ejemplo muestra una disminución notable en la duración del trabajo. Esto se debe a que el grupo se configuró con cuatro tareas por nodo, lo que permite la ejecución en paralelo de tareas de forma que el trabajo se completa en casi una cuarta parte del tiempo.

```
Nodes: 1
Node size: large
Task slots per node: 4
Max slots per task: 1
Tasks: 32
Duration: 00:08:48.2423500
```

## <a name="next-steps"></a>Pasos siguientes

- Pruebe el mapa térmico de [Batch Explorer](https://azure.github.io/BatchExplorer/). Batch Explorer es una herramienta de cliente independiente, completa y gratuita que puede ayudarle a crear, depurar y supervisar aplicaciones de Azure Batch. Cuando ejecuta la aplicación de ejemplo [ParallelTasks](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ParallelTasks), la característica Mapa térmico de Batch Explorer le permite ver fácilmente la ejecución de tareas paralelas en cada nodo.
- Explore [ejemplos de Azure Batch en GitHub](https://github.com/Azure/azure-batch-samples).
- Obtenga más información sobre las [dependencias de las tareas de Batch](batch-task-dependencies.md).
