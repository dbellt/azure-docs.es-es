---
title: Ejecución de cargas de trabajo en máquinas virtuales rentables de prioridad baja
description: Aprenda a aprovisionar máquinas virtuales de prioridad baja para reducir el costo de las cargas de trabajo de Azure Batch.
author: mscurrell
ms.topic: how-to
ms.date: 03/03/2021
ms.custom: seodec18
ms.openlocfilehash: d6e6277292d12de10ad55c5f7d912f41e92b5b18
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107989313"
---
# <a name="use-low-priority-vms-with-batch"></a>Uso de máquinas virtuales de prioridad baja con Batch

Azure Batch ofrece máquinas virtuales (VM) de prioridad baja para reducir el costo de las cargas de trabajo de Batch. Las máquinas virtuales de prioridad baja posibilitan nuevos tipos de cargas de trabajo de Batch al permitir el uso de una elevada capacidad de proceso a un costo muy reducido.

Las máquinas virtuales de prioridad baja aprovechan la capacidad sobrante en Azure. Al especificar máquinas virtuales de prioridad baja en sus grupos, Azure Batch puede usar este excedente cuando esté disponible.

El inconveniente del uso de máquinas virtuales de prioridad baja es que esas máquinas virtuales pueden no estar siempre disponibles para su asignación o pueden reemplazarse en cualquier momento, según la capacidad disponible. Por este motivo, las máquinas virtuales de prioridad baja resultan más adecuadas para las cargas de trabajo de procesamiento por lotes y asincrónicas en las que el tiempo de finalización del trabajo es flexible y el trabajo se distribuye entre muchas máquinas virtuales.

Las máquinas virtuales de prioridad baja se ofrecen a un precio considerablemente reducido en comparación con las máquinas virtuales dedicadas. Para más información sobre precios, consulte [Precios de Batch](https://azure.microsoft.com/pricing/details/batch/).

> [!NOTE]
> [Las máquinas virtuales de Spot](https://azure.microsoft.com/pricing/spot/) ahora están disponibles para [máquinas virtuales de instancia única](../virtual-machines/spot-vms.md) y [conjuntos de escalado de máquinas virtuales](../virtual-machine-scale-sets/use-spot.md). Las máquinas virtuales de Spot son una evolución de las máquinas virtuales de prioridad baja, pero difieren en que los precios pueden variar y se puede establecer un precio máximo opcional al asignar máquinas virtuales de Spot.
>
>Los grupos de Azure Batch comenzarán a admitir VM de Spot en el futuro, con nuevas versiones de las [API y las herramientas de Batch](./batch-apis-tools.md). Una vez que las VM de Spot estén disponibles, las VM de baja prioridad quedarán en desuso, si bien seguirán siendo compatibles durante al menos 12 meses si se usan las versiones actuales de la API y las herramientas. Así habrá tiempo suficiente para realizar la migración a las VM de Spot.
>
> Solo se admitirán VM de Spot para los grupos de configuración de máquina virtual. Para usar las VM de Spot, los grupos de configuración de Cloud Services tendrán que [migrarse a grupos de configuración de máquina virtual](batch-pool-cloud-service-to-virtual-machine-configuration.md).

## <a name="batch-support-for-low-priority-vms"></a>Compatibilidad de Batch con máquinas virtuales de prioridad baja

Azure Batch ofrece varias funcionalidades que facilitan el consumo y que se benefician de las máquinas virtuales de prioridad baja:

- Los grupos de Batch pueden contener tanto máquinas virtuales dedicadas como máquinas virtuales de prioridad baja. El número de cada tipo de máquina virtual se puede especificar cuando se crea un grupo, o puede cambiarse en cualquier momento para un grupo ya existente, mediante la operación explícita de cambio de tamaño o el escalado automático. El envío de trabajos y tareas puede permanecer sin cambios independientemente de los tipos de máquina virtual en el grupo. También puede configurar un grupo para usar completamente máquinas virtuales de prioridad baja para ejecutar trabajos lo más barato posible, pero volver a máquinas virtuales dedicadas si la capacidad desciende por debajo de un umbral mínimo, a fin de mantener los trabajos en funcionamiento.
- Los grupos de Batch buscan automáticamente el número objetivo de máquinas virtuales de prioridad baja. Si las máquinas virtuales se reemplazan o no están disponibles, Batch intenta reemplazar la capacidad perdida y volver al destino.
- Cuando se interrumpen las tareas, Batch lo detecta y vuelve a poner las tareas en cola para que se ejecuten nuevamente.
- Las máquinas virtuales de prioridad baja tienen una cuota de vCPU diferente de las máquinas virtuales dedicadas. La cuota para las máquinas virtuales de prioridad baja es mayor que la de las máquinas virtuales dedicadas, ya que las primeras tienen un precio inferior. Para más información, consulte [Límites y cuotas del servicio Batch](batch-quota-limit.md#resource-quotas).

> [!NOTE]
> Las máquinas virtuales de prioridad baja no se admiten actualmente para cuentas de Batch creadas en [modo de suscripción de usuario](accounts.md).

## <a name="considerations-and-use-cases"></a>Consideraciones y casos de uso

Muchas cargas de trabajo de Batch son una buena opción para las máquinas virtuales de prioridad baja. Considere la posibilidad de usarlas cuando los trabajos estén divididos en muchas tareas paralelas o cuando haya muchos trabajos escalados horizontalmente y distribuidos en muchas máquinas virtuales.

Algunos ejemplos de casos de uso de procesamiento por lotes adecuados para usar máquinas virtuales de prioridad baja son:

- **Desarrollo y pruebas**: en concreto, si se están desarrollando soluciones a gran escala se pueden obtener ahorros significativos. Aunque todos los tipos de pruebas pueden beneficiarse, será especialmente ventajoso para pruebas de carga y regresión a gran escala.
- **Capacidad a petición complementaria**: las máquinas virtuales de prioridad baja pueden usarse como complemento a las máquinas virtuales dedicadas normales. Cuando están disponibles, los trabajos pueden escalarse y, por lo tanto, completarse más rápido con un costo menor; cuando no están disponibles, la línea de base de las máquinas virtuales dedicadas sigue estando disponible.
- **Tiempo de ejecución flexible de los trabajos**: si hay flexibilidad en el tiempo en que tienen que completarse los trabajos, entonces se pueden tolerar posibles bajadas en la capacidad; sin embargo, con la adición de VM de prioridad baja, los trabajos se ejecutan con frecuencia más rápido y con un costo menor.

Los grupos de Batch se pueden configurar para usar máquinas virtuales de prioridad baja de varias maneras:

- Un grupo puede usar solo máquinas virtuales de prioridad baja. En este caso, Batch recupera cualquier capacidad reemplazada cuando esté disponible. Esta configuración es la manera más barata de ejecutar trabajos.
- Las máquinas virtuales de prioridad baja se pueden usar conjuntamente con una línea de base fija de máquinas virtuales dedicadas. El número fijo de máquinas virtuales dedicadas garantiza que siempre haya algo de capacidad para mantener el progreso de un trabajo.
- Un grupo puede usar una combinación dinámica de máquinas virtuales dedicadas y de prioridad baja, de modo que las máquinas virtuales de prioridad baja más baratas se usen únicamente cuando estén disponibles, pero las máquinas virtuales dedicadas de precio completo se escalen verticalmente cuando sea necesario. Esta configuración mantiene una cantidad mínima de capacidad disponible para mantener el progreso de los trabajos.

Tenga en cuenta lo siguiente al planear el uso de máquinas virtuales de prioridad baja:

- Para maximizar el uso de la capacidad sobrante en Azure, los trabajos adecuados se pueden escalar horizontalmente.
- Puede que, en ocasiones, las máquinas virtuales no estén disponibles o se reemplacen; como resultado, la capacidad para los trabajos se ve mermada y puede generar interrupciones de las tareas o la necesidad de ejecutarlas una y otra vez.
- Las tareas con tiempos de ejecución más cortos tienden a funcionar mejor con máquinas virtuales de prioridad baja. Los trabajos con tareas más largas pueden resultar más afectados si se interrumpen. Si las tareas de ejecución prolongada implementan puntos de control para guardar el progreso mientras se ejecutan, este impacto puede reducirse. 
- Los trabajos de MPI de larga ejecución que usan varias máquinas virtuales no son adecuados para usar máquinas virtuales de prioridad baja, porque una máquina virtual reemplazada probablemente ocasione que todo el trabajo tenga que ejecutarse de nuevo.
- Los nodos de prioridad baja pueden marcarse como inutilizables si las [reglas del grupo de seguridad de red (NSG)](batch-virtual-network.md#network-security-groups-specifying-subnet-level-rules) están configuradas incorrectamente.

## <a name="create-and-manage-pools-with-low-priority-vms"></a>Creación y administración de grupos con máquinas virtuales de prioridad baja

Un grupo de Batch puede contener tanto máquinas virtuales dedicadas como de prioridad baja (también conocidas como nodos de proceso). Puede establecer el número objetivo de nodos de proceso tanto para las máquinas virtuales dedicadas como para las de prioridad baja. El número de nodos objetivo especifica el número de máquinas virtuales que quiere tener en el grupo.

Por ejemplo, para crear un grupo con máquinas virtuales de Azure (en este caso, máquinas virtuales Linux) con un destino de 5 máquinas virtuales dedicadas y 20 máquinas virtuales de prioridad baja:

```csharp
ImageReference imageRef = new ImageReference(
    publisher: "Canonical",
    offer: "UbuntuServer",
    sku: "16.04-LTS",
    version: "latest");

// Create the pool
VirtualMachineConfiguration virtualMachineConfiguration =
    new VirtualMachineConfiguration("batch.node.ubuntu 16.04", imageRef);

pool = batchClient.PoolOperations.CreatePool(
    poolId: "vmpool",
    targetDedicatedComputeNodes: 5,
    targetLowPriorityComputeNodes: 20,
    virtualMachineSize: "Standard_D2_v2",
    virtualMachineConfiguration: virtualMachineConfiguration);
```

Puede obtener el número actual de nodos tanto para las máquinas virtuales dedicadas como para las de prioridad baja:

```csharp
int? numDedicated = pool1.CurrentDedicatedComputeNodes;
int? numLowPri = pool1.CurrentLowPriorityComputeNodes;
```

Los nodos de grupo tienen una propiedad que indica si el nodo es una máquina virtual dedicada o de prioridad baja:

```csharp
bool? isNodeDedicated = poolNode.IsDedicated;
```

En ocasiones, es posible que las máquinas virtuales se reemplacen. Cuando esto suceda, las tareas que se estuvieran ejecutando en las máquinas virtuales del nodo reemplazado se pondrían de nuevo en cola y se volverían a ejecutar.

En el caso de los grupos de configuración de máquinas virtuales, Batch también realiza lo siguiente:

- Las máquinas virtuales reemplazadas tienen su estado actualizado a **Reemplazada**. 
- La máquina virtual se elimina de manera eficaz, lo que hace que se pierdan los datos almacenados localmente en ella.
- Una operación de enumeración de nodos del grupo sigue devolviendo los nodos reemplazados.
- El grupo intenta continuamente alcanzar el número objetivo de nodos de prioridad baja disponibles. Cuando se encuentra la capacidad de reemplazo, los nodos mantienen sus identificaciones, pero se reinicializan, pasando por los estados **Creando** e **Iniciando** antes de que estén disponibles para la programación de tareas.
- Los recuentos de reemplazos están disponibles como una métrica en el portal de Azure.

## <a name="scale-pools-containing-low-priority-vms"></a>Escalado de grupos que contienen máquinas virtuales de prioridad baja

Al igual que sucede con grupos que se componen únicamente de máquinas virtuales dedicadas, es posible escalar un grupo que contenga máquinas virtuales de prioridad baja mediante la llamada al método Resize o por medio de escalado automático.

La operación de cambio de tamaño de grupo tiene un segundo parámetro opcional que actualiza el valor de **targetLowPriorityNodes**:

```csharp
pool.Resize(targetDedicatedComputeNodes: 0, targetLowPriorityComputeNodes: 25);
```

La fórmula de escalado automático de grupo admite máquinas virtuales de prioridad baja del siguiente modo:

- Puede obtener o establecer el valor de la variable definida por el servicio **$TargetLowPriorityNodes**.
- Puede obtener el valor de la variable definida por el servicio **$CurrentLowPriorityNodes**.
- Puede obtener el valor de la variable definida por el servicio **$PreemptedNodeCount**. Esta variable devuelve el número de nodos en estado reemplazado y le permite escalar o reducir verticalmente el número de nodos dedicados, en función del número de nodos reemplazados que no estén disponibles.

## <a name="configure-jobs-and-tasks"></a>Configuración de trabajos y tareas

Los trabajos y las tareas requieren poca configuración adicional de los nodos de prioridad baja. Tenga en cuenta lo siguiente:

- La propiedad JobManagerTask de un trabajo tiene una propiedad **AllowLowPriorityNode**. Cuando esta propiedad es true, la tarea del administrador de trabajos se puede programar tanto en un nodo dedicado como de prioridad baja. Si es false, la tarea del administrador de trabajos se programa solo en un nodo dedicado.
- La [variable de entorno](batch-compute-node-environment-variables.md) AZ_BATCH_NODE_IS_DEDICATED está disponible para una aplicación de tareas de modo que pueda determinar si se ejecuta en un nodo de prioridad baja o dedicado.

## <a name="view-metrics-for-low-priority-vms"></a>Visualización de métricas de máquinas virtuales de baja prioridad

Hay nuevas métricas disponibles en [Azure Portal ](https://portal.azure.com) para los nodos de baja prioridad. Estas son las métricas:

- Recuento de nodos de baja prioridad
- Recuento de núcleos de baja prioridad
- Recuento de nodos con prioridad

Para ver estas métricas en Azure Portal

1. Vaya a la cuenta de Batch en Azure Portal.
2. Seleccione **Métricas** en la sección **Supervisión**.
3. Seleccione las métricas que quiere en la lista **Métricas**.

## <a name="next-steps"></a>Pasos siguientes

- Conozca el [flujo de trabajo y los recursos principales del servicio Batch](batch-service-workflow-features.md), como grupos, nodos, trabajos y tareas.
- Obtenga información acerca de las [API y herramientas de Batch](batch-apis-tools.md) disponibles para la creación de soluciones de Batch.
- Comience a planear la migración de máquinas virtuales de prioridad baja a máquinas virtuales de Spot. Si usa máquinas virtuales de prioridad baja con grupos de **configuración de Cloud Services** (que están [en desuso](https://azure.microsoft.com/updates/azure-batch-cloudserviceconfiguration-pools-will-be-retired-on-29-february-2024/)), planee migrar a [grupos de **Configuración de máquina virtual**](nodes-and-pools.md#configurations) en su lugar.
