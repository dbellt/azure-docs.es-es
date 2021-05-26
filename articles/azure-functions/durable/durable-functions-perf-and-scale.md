---
title: 'Rendimiento y escalado horizontal en Durable Functions: Azure'
description: Obtenga información sobre las características de escalado únicas de la extensión Durable Functions para Azure Functions.
author: cgillum
ms.topic: conceptual
ms.date: 05/13/2021
ms.author: azfuncdf
ms.openlocfilehash: a91baf110e08794bcf2dedec2a61f0e6c2c734a5
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110375883"
---
# <a name="performance-and-scale-in-durable-functions-azure-functions"></a>Rendimiento y escalado horizontal en Durable Functions (Azure Functions)

Para optimizar el rendimiento y la escalabilidad, es importante comprender las características únicas de escalado de [Durable Functions](durable-functions-overview.md).

## <a name="azure-storage-provider"></a>Proveedor de Azure Storage

La configuración predeterminada de Durable Functions almacena este estado del entorno de ejecución en una cuenta (clásica) de Azure Storage. Toda la ejecución de funciones la realizan colas de Azure Storage. La orquestación, el estado de la entidad y el historial se almacenan en tablas de Azure. Los blobs de Azure y las concesiones de blobs se usan para distribuir instancias y entidades de orquestación entre varias instancias de aplicación (también conocidas como *trabajos* o sencillamente *máquinas virtuales*). En esta sección se explica con más detalle los distintos artefactos de Azure Storage y cómo afectan al rendimiento y la escalabilidad.

> [!NOTE]
> Este documento se centra principalmente en las características de rendimiento y escalabilidad de Durable Functions mediante el proveedor de Azure Storage predeterminado. Sin embargo, también están disponibles otros proveedores de almacenamiento. Para más información sobre los proveedores de almacenamiento compatibles con Durable Functions y poder compararlos, consulte la documentación sobre [proveedores de almacenamiento de Durable Functions](durable-functions-storage-providers.md).

### <a name="history-table"></a>Tabla del historial

La tabla **History** es una tabla de Azure Storage que contiene los eventos del historial de todas las instancias de orquestación dentro de una central de tareas. El nombre de esta tabla está en formato *TaskHubName* History. Según se ejecutan las instancias, se van agregando nuevas filas a esta tabla. La clave de partición de esta tabla proviene del identificador de instancia de la orquestación. Los identificadores de instancia son aleatorios de manera predeterminada, lo que garantiza una distribución óptima de las particiones internas en Azure Storage. La clave de fila de esta tabla es un número de secuencia que se usa para ordenar los eventos de historial.

Cuando es necesario ejecutar una instancia de orquestación, las filas correspondientes de la tabla History se cargan en la memoria mediante una consulta por rango dentro de una sola partición de tabla. Estos *eventos del historial* se reproducen luego en el código de función de orquestación para regresar a su estado de punto de control previo. El uso del historial de ejecución para reconstruir el estado de esta manera se ve influenciado por el [patrón Event Sourcing](/azure/architecture/patterns/event-sourcing).

> [!TIP]
> Los datos de orquestación almacenados en la tabla History incluyen cargas de salida de las funciones de actividad y de suborquestador. Las cargas de eventos externos también se almacenan en la tabla History. Dado que el historial completo se carga en la memoria cada vez que un orquestador necesita ejecutarse, un historial lo suficientemente grande puede provocar una presión de memoria significativa en una máquina virtual determinada. La longitud y el tamaño del historial de orquestación se pueden reducir dividiendo las orquestaciones grandes en varias suborquestaciones o reduciendo el tamaño de las salidas devueltas por las funciones de actividad y de suborquestador a las que llama. Como alternativa, puede reducir el uso de memoria disminuyendo las [limitaciones de simultaneidad](#concurrency-throttles) por máquina virtual para limitar el número de orquestaciones que se cargan en la memoria de manera simultánea.

### <a name="instances-table"></a>Tabla Instances

La tabla **Instances** contiene los estados de todas las instancias de orquestación y entidad dentro de una central de tareas. Según se crean las instancias, se van agregando nuevas filas a esta tabla. La clave de partición de esta tabla es el identificador de instancia de orquestación o la clave de entidad, y la clave de fila es una cadena vacía. Hay una fila por cada instancia de orquestación o entidad.

Esta tabla se utiliza para satisfacer las [solicitudes de consulta de instancia de código](durable-functions-instance-management.md#query-instances) así como las llamadas de [API HTTP de consulta de estado](durable-functions-http-api.md#get-instance-status). Se mantiene coherente en última instancia con el contenido de la tabla **History** mencionada anteriormente. El uso de una tabla de Azure Storage independiente para satisfacer con eficiencia operaciones de consulta de instancias de esta manera se ve influenciada por el [patrón Command and Query Responsibility Segregation (CQRS)](/azure/architecture/patterns/cqrs).

> [!TIP]
> La creación de particiones de la tabla *Instances* permite almacenar millones de instancias de orquestación sin ningún impacto perceptible en el rendimiento o la escala del entorno de ejecución. Sin embargo, el número de instancias puede tener un impacto significativo en el rendimiento de las [consultas de instancias múltiples](durable-functions-instance-management.md#query-all-instances). Para controlar la cantidad de datos almacenados en estas tablas, considere la posibilidad de [purgar periódicamente los datos de instancia antiguos](durable-functions-instance-management.md#purge-instance-history).

### <a name="internal-queue-triggers"></a>Desencadenadores de cola interna

Las funciones de orquestador, entidad y actividad se desencadenan mediante colas internas de la central de tareas de la aplicación de funciones. El uso de colas de este modo proporciona garantías de entrega confiable de mensajes "al menos una vez". Hay dos tipos de colas en Durable Functions: **de control** y **de elementos de trabajo**.

#### <a name="the-work-item-queue"></a>Cola de elementos de trabajo

En Durable Functions hay una cola de elementos de trabajo por central de tareas. Se trata de una cola básica y se comporta del mismo modo que cualquier otra cola `queueTrigger` en Azure Functions. Esta cola se usa para desencadenar *funciones de actividad* sin estado al eliminar de la cola un único mensaje a la vez. Cada uno de estos mensajes contiene entradas de la función de actividad y metadatos adicionales, como qué función se debe ejecutar. Cuando una aplicación de Durable Functions admita el escalado horizontal de varias máquinas virtuales, estas compiten para adquirir las tareas de la cola de elementos de trabajo.

#### <a name="control-queues"></a>Colas de control

Hay varias *colas de control* por central de tareas en Durable Functions. Una *cola de control* es más sofisticada que una cola de elementos de trabajo, más sencilla. Las colas de control se usan para desencadenar las funciones de orquestador y entidad con estado. Dado que las instancias de la función de orquestador y de entidad son singletons con estado, es importante que cada orquestación o entidad solo se procese por un trabajo a la vez. Para lograr esta restricción, cada instancia o entidad de orquestación se asigna a una única cola de control. Estas colas de control tienen equilibrio de carga entre los trabajos para garantizar que cada una de ellas se procese únicamente por un trabajo cada vez. Encontrará más detalles sobre este comportamiento en las secciones siguientes.

Las colas de control contienen una gran variedad de tipos de mensajes del ciclo de vida de la orquestación. Algunos ejemplos son los [mensajes de control del orquestador](durable-functions-instance-management.md), los mensajes de *respuesta* de la función de actividad y los del temporizador. Como máximo, se pueden eliminar 32 mensajes de una cola de control en un único sondeo. Estos mensajes contienen datos de carga, así como metadatos, incluido para qué instancia de orquestación están previstos. Si hay varios mensajes eliminados de la cola previstos para la misma instancia de orquestación, se procesarán como lote.

Los mensajes de la cola de control se sondean constantemente mediante un subproceso en segundo plano. El tamaño de lote de los sondeos de cola se controla mediante el valor `controlQueueBatchSize` de host.jsen y tiene un valor predeterminado de 32 (el valor máximo admitido por colas de Azure). El número máximo de mensajes de la cola de control capturados previamente almacenados en un búfer de memoria se controla mediante el valor `controlQueueBufferThreshold` de host.jsen. El valor predeterminado de `controlQueueBufferThreshold` varía en función de una serie de factores, como el tipo de plan de hospedaje. Para obtener más información sobre esta configuración, consulte la documentación del [esquema de host.js](../functions-host-json.md#durabletask).

> [!TIP]
> Aumentar el valor de `controlQueueBufferThreshold` permite que una sola orquestación o entidad procese eventos más rápidamente. Sin embargo, aumentar este valor también puede dar lugar a un uso de memoria más elevado. El aumento del uso de memoria se debe en parte a la extracción de más mensajes de la cola y, en parte, a la captura de más historiales de orquestación en la memoria. Por lo tanto, disminuir el valor de `controlQueueBufferThreshold` puede ser una manera eficaz de reducir el uso de memoria.

#### <a name="queue-polling"></a>Sondeo de cola

La extensión Durable Task implementa un algoritmo de interrupción exponencial aleatorio para reducir el efecto del sondeo de cola inactiva en los costos de transacción de almacenamiento. Cuando encuentra un mensaje, el entorno de ejecución busca inmediatamente otro mensaje. Si no encuentra ningún mensaje, espera un intervalo de tiempo concreto antes de volver a intentarlo. Después de varios intentos fallidos para obtener un mensaje de la cola, el tiempo de espera sigue aumentando hasta que alcanza el tiempo de espera máximo, predeterminado en 30 segundos.

El retraso de sondeo máximo se configura mediante la propiedad `maxQueuePollingInterval` en el [archivo host.json](../functions-host-json.md#durabletask). El establecimiento de esta propiedad en un valor más alto puede provocar mayores latencias de procesamiento de mensajes. Las latencias elevadas solo se pueden esperar después de períodos de inactividad. El establecimiento de esta propiedad en un valor más bajo puede provocar [costos de almacenamiento más elevados](durable-functions-billing.md#azure-storage-transactions) debido al mayor número de transacciones de almacenamiento.

> [!NOTE]
> Cuando se ejecuta en los planes Consumo y Premium de Azure Functions, el [controlador de escala de Azure Functions](../event-driven-scaling.md) sondeará cada cola de control y elemento de trabajo una vez cada 10 segundos. Este sondeo adicional es necesario para determinar cuándo activar instancias de aplicaciones de función y tomar decisiones de escalado. En el momento de escribir este artículo, este intervalo de 10 segundos es constante y no se puede configurar.

#### <a name="orchestration-start-delays"></a>Retrasos en el inicio de la orquestación
Las instancias de las orquestaciones se inician con la inserción de un mensaje `ExecutionStarted` en una de las colas de control de la central de tareas. En determinadas condiciones, se pueden observar retrasos de varios segundos entre el momento en que se programa la ejecución de una orquestación y el momento en el que se inicia la ejecución. Durante este intervalo de tiempo, la instancia de la orquestación permanece en estado `Pending`. Hay dos causas posibles de este retraso:

1. **Colas de control pendientes**: si la cola de control de esta instancia contiene un gran número de mensajes, puede tardar un tiempo antes de que el entorno de ejecución reciba y procese el mensaje `ExecutionStarted`. Los trabajos pendientes de mensajes se pueden producir cuando las orquestaciones procesan muchos eventos simultáneamente. Entre los eventos que van a la cola de control se incluyen los eventos de inicio de orquestación, finalización de actividad, temporizadores duraderos, terminación y eventos externos. Si este retraso se produce en circunstancias normales, considere la posibilidad de crear una nueva central de tareas con un mayor número de particiones. La configuración de más particiones hará que el entorno de ejecución cree más colas de control para la distribución de la carga. Cada partición se corresponde con una cola de control con una relación de 1:1, con un máximo de 16 particiones.

2. **Retrasos en los sondeos de retroceso**: otra causa común de los retrasos de la orquestación es el [comportamiento de los sondeos de retroceso que se ha descrito anteriormente para las colas de control](#queue-polling). Sin embargo, este retraso solo se espera cuando una aplicación se escala horizontalmente a dos o más instancias. Si solo hay una instancia de la aplicación o si la instancia de la aplicación que inicia la orquestación también es la misma instancia que sondea la cola de control de destino, no habrá un retraso en el sondeo de la cola. Los retrasos de los sondeos de retroceso se pueden reducir mediante la actualización de las opciones del archivo **host.json**, tal y como se ha descrito anteriormente.

### <a name="storage-account-selection"></a>Selección de una cuenta de almacenamiento

Las colas, tablas y blobs usados por Durable Functions se crean en una cuenta de Azure Storage configurada. Se puede especificar la cuenta que se va a usar mediante el valor `durableTask/storageProvider/connectionStringName` (o `durableTask/azureStorageConnectionStringName` en Durable Functions 1.x) del archivo **host.json**.

#### <a name="durable-functions-2x"></a>Durable Functions 2.x

```json
{
  "extensions": {
    "durableTask": {
      "storageProvider": {
        "connectionStringName": "MyStorageAccountAppSetting"
      }
    }
  }
}
```

#### <a name="durable-functions-1x"></a>Durable Functions 1.x

```json
{
  "extensions": {
    "durableTask": {
      "azureStorageConnectionStringName": "MyStorageAccountAppSetting"
    }
  }
}
```

Si no se especifica, se utiliza la cuenta de almacenamiento `AzureWebJobsStorage` predeterminada. Para las cargas de trabajo sensibles al rendimiento, se recomienda, sin embargo, configurar una cuenta de almacenamiento que no sea la predeterminada. Durable Functions utiliza Azure Storage de forma intensiva y el uso de una cuenta de almacenamiento dedicada aísla el uso del almacenamiento de Durable Functions del uso interno por parte del host de Azure Functions.

> [!NOTE]
> Las cuentas estándar de uso general de Azure Storage son necesarias cuando se usa el proveedor de Azure Storage. No se admite ningún otro tipo de cuenta de almacenamiento. Se recomienda encarecidamente usar cuentas de almacenamiento de uso general v1 heredadas para Durable Functions. Las cuentas de almacenamiento v2 más recientes pueden ser mucho más caras para cargas de trabajo de Durable Functions. Para obtener más información sobre los tipos de cuentas de Azure Storage, consulte la documentación [general acerca de la cuenta de Azure Storage](../../storage/common/storage-account-overview.md).

### <a name="orchestrator-scale-out"></a>Escalabilidad horizontal del orquestador

Aunque las funciones de actividad se pueden escalar horizontalmente de forma infinita mediante la adición de más máquinas virtuales elásticamente, las instancias y entidades de orquestador individuales están limitadas para habitar en una única partición, y el número máximo de particiones está limitado por el valor `partitionCount` de su `host.json`. 

> [!NOTE]
> En general, las funciones de orquestador se han diseñado para que sean ligeras y no necesiten grandes capacidades de computación. Por tanto, no es necesario crear un gran número de particiones de cola de control para obtener un rendimiento excelente para las orquestaciones. La mayor parte del trabajo pesado se realiza en las funciones de actividad sin estado, que se pueden escalar en horizontal infinitamente.

El número de colas de control se define en el archivo **host.json**. El siguiente fragmento de código de host.json de ejemplo establece la propiedad `durableTask/storageProvider/partitionCount` (o `durableTask/partitionCount` en Durable Functions 1.x) en `3`. Tenga en cuenta que hay tantas colas de control como particiones.

#### <a name="durable-functions-2x"></a>Durable Functions 2.x

```json
{
  "extensions": {
    "durableTask": {
      "storageProvider": {
        "partitionCount": 3
      }
    }
  }
}
```

#### <a name="durable-functions-1x"></a>Durable Functions 1.x

```json
{
  "extensions": {
    "durableTask": {
      "partitionCount": 3
    }
  }
}
```

Una central de tareas se puede configurar con entre 1 y 16 particiones. Si no se especifica, el número de participaciones predeterminado es **4**.

Durante los escenarios de bajo tráfico, la aplicación se reducirá horizontalmente, por lo que las particiones serán administradas por un pequeño número de trabajos. Como ejemplo, considere el siguiente diagrama.

![Diagrama de orquestaciones de reducción horizontal](./media/durable-functions-perf-and-scale/scale-progression-1.png)

En el diagrama anterior, vemos que los orquestadores del 1 al 6 tienen equilibrio de carga entre las particiones. Del mismo modo, las particiones, como las actividades, tienen equilibrio de carga entre los trabajos. Las particiones tienen equilibrio de carga entre los trabajos, independientemente del número de orquestadores que se inicien.

Si está ejecutando los planes Consumo o Elástico Premium de Azure Functions, o si tiene configurado el escalado automático basado en la carga, se asignarán más trabajos a medida que aumente el tráfico y las particiones acabarán por equilibrar la carga entre todos los trabajos. Si continuamos escalando horizontalmente, cada partición se administrará finalmente por un único trabajo. Por otro lado, las actividades seguirán teniendo un equilibrio de carga en todos los trabajos. Esto se puede observar en la siguiente imagen.

![Primer diagrama de orquestaciones escaladas horizontalmente](./media/durable-functions-perf-and-scale/scale-progression-2.png)

El límite superior del número máximo de orquestaciones simultáneas _activas_ en *un momento dado* es igual al número de trabajos asignados a su aplicación _por_ su valor para `maxConcurrentOrchestratorFunctions`. Este límite superior puede ser más preciso cuando las particiones se escalan de manera horizontal por completo entre los trabajos. Cuando se hayan escalado por completo de manera horizontal, y dado que cada trabajo tendrá solo una instancia de host de Functions, el número máximo de instancias de orquestador simultáneas _activas_ será igual al número de particiones _por_ su valor para `maxConcurrentOrchestratorFunctions`.

> [!NOTE]
> En este contexto, *activo* significa que una orquestación o entidad se carga en la memoria y que procesa *nuevos eventos*. Si la orquestación o entidad está esperando más eventos, como el valor devuelto de una función de actividad, se descarga de la memoria y ya no se considera *activa*. Las orquestaciones y entidades se volverán a cargar posteriormente en la memoria solo cuando haya nuevos eventos que procesar. No existe un número máximo práctico de orquestaciones o entidades *totales* que se puedan ejecutar en una sola máquina virtual, incluso si todas están en estado "En ejecución". La única limitación es el número de instancias de entidad o orquestación *activas simultáneamente*.

En la imagen siguiente se muestra un escenario con un escalado horizontal completo en el que se agregan más orquestadores pero algunos están inactivos (en gris).

![Segundo diagrama de orquestaciones escaladas horizontalmente](./media/durable-functions-perf-and-scale/scale-progression-3.png)

Durante el escalado horizontal, se pueden redistribuir las concesiones de la cola de control entre las instancias de host de Functions para asegurarse de que las particiones se distribuyen uniformemente. Estas concesiones se implementan internamente como concesiones de almacenamiento de blobs de Azure y garantizan que solo se ejecuta una instancia o entidad de orquestación individual en una única instancia de host a la vez. Si una central de tareas está configurada con tres particiones (y, por lo tanto, tres colas de control), las entidades y las instancias de orquestación pueden tener equilibrio de carga en las tres instancias de host que albergan las concesiones. Se pueden agregar máquinas virtuales adicionales para aumentar la capacidad de ejecución de la función de actividad,

El siguiente diagrama ilustra cómo interactúa el host de Azure Functions con las entidades de almacenamiento en un entorno de escalado horizontal.

![Diagrama de escalado](./media/durable-functions-perf-and-scale/scale-interactions-diagram.png)

Tal como se muestra en el diagrama anterior, todas las máquinas virtuales compiten por los mensajes en la cola de elementos de trabajo. Sin embargo, solo tres de ellas pueden adquirir mensajes de las colas de control y cada una bloquea una sola cola de control.

Las instancias de orquestación y entidades se distribuyen entre todas las instancias de la cola de control. La distribución se realiza al aplicar un algoritmo hash al identificador de instancia de la orquestación o el par de nombre y clave de la entidad. Los identificadores de instancia de orquestación son, de forma predeterminada, GUID aleatorios que aseguran que las instancias se distribuyen equitativamente entre todas las colas de control.

En general, las funciones de orquestador se han diseñado para que sean ligeras y no necesiten grandes capacidades de computación. Por tanto, no es necesario crear un gran número de particiones de cola de control para obtener un rendimiento excelente para las orquestaciones. La mayor parte del trabajo pesado se realiza en las funciones de actividad sin estado, que se pueden escalar en horizontal infinitamente.

### <a name="auto-scale"></a>Escalado automático

Al igual que todas las instancias de Azure Functions que se ejecutan en los planes Consumo y Elástico Premium, Durable Functions admite el escalado automático a través del [controlador de escalado de Azure Functions](../event-driven-scaling.md#runtime-scaling). El controlador de escalado supervisa la latencia de todas las colas emitiendo periódicamente comandos _peek_. En función de las latencias de los mensajes inspeccionados, el controlador de escalado decidirá si desea agregar o quitar máquinas virtuales.

Si el controlador de escalado determina que las latencias de mensaje de la cola de control son demasiado altas, agregará instancias de máquina virtual hasta que la latencia de mensajes se reduzca a un nivel aceptable o alcance el número de particiones de la cola de control. De igual forma, el controlador de escalado va a agregar continuamente instancias de máquina virtual si las latencias de la cola de elementos de trabajo son altas, independientemente del número de particiones.

> [!NOTE]
> A partir de Durable Functions 2.0, las aplicaciones de función se pueden configurar de modo que se ejecuten en los puntos de conexión de servicio protegidos mediante red virtual en el plan Elástico Premium. En esta configuración, los desencadenadores de Durable Functions inician solicitudes de escalado en lugar del controlador de escalado. Para obtener más información, consulte [Supervisión de escala en entorno de ejecución](../functions-networking-options.md#premium-plan-with-virtual-network-triggers).

## <a name="thread-usage"></a>Uso de subprocesos

Las funciones de orquestador se ejecutan en un solo subproceso para asegurarse de que la ejecución pueda ser determinista entre muchas de las reproducciones. Debido a esta ejecución en un único subproceso, es importante que los subprocesos de las funciones de orquestador no realicen tareas de uso intensivo de CPU, ni operaciones de E/S ni bloqueos por ningún motivo. Cualquier trabajo que requiera operaciones de E/S, bloqueos o varios subprocesos debe derivarse a las funciones de actividad.

Las funciones de actividad tienen el mismo comportamiento que las funciones normales desencadenadas por colas. Pueden realizar operaciones de E/S, ejecutar operaciones de uso intensivo de CPU y utilizar varios subprocesos con seguridad. Dado que los desencadenadores de actividad no tienen estado, se escalan horizontalmente a un número ilimitado de máquinas virtuales sin problema.

Las funciones de entidad también se ejecutan en un único subproceso y las operaciones se procesan de una en una. Pero las funciones de entidad no tienen ninguna restricción en cuanto al tipo de código que se puede ejecutar.

## <a name="concurrency-throttles"></a>Limitaciones de simultaneidad

Azure Functions admite la ejecución de varias funciones simultáneamente dentro de una instancia de aplicación única. Esta ejecución simultánea ayuda a aumentar el paralelismo y minimiza el número de "arranques en frío" que experimentará una aplicación típica a lo largo del tiempo. Pero la alta simultaneidad puede agotar los recursos del sistema por máquina virtual, como las conexiones de red o la memoria disponible. Según las necesidades de la aplicación de función, puede ser necesario limitar la simultaneidad por instancia para evitar la posibilidad de quedarse sin memoria en situaciones de carga alta.

Los límites de simultaneidad de las funciones de actividad, orquestador y entidad se pueden configurar en el archivo **host.json**. La configuración relevante es `durableTask/maxConcurrentActivityFunctions` para las funciones de actividad y `durableTask/maxConcurrentOrchestratorFunctions` para las funciones de orquestador y de entidad. Esta configuración controla el número máximo de funciones de orquestador, entidad o actividad que se pueden cargar simultáneamente en la memoria.

### <a name="functions-20"></a>Functions 2.0

```json
{
  "extensions": {
    "durableTask": {
      "maxConcurrentActivityFunctions": 10,
      "maxConcurrentOrchestratorFunctions": 10
    }
  }
}
```

### <a name="functions-1x"></a>Functions 1.x

```json
{
  "durableTask": {
    "maxConcurrentActivityFunctions": 10,
    "maxConcurrentOrchestratorFunctions": 10
  }
}
```

En el ejemplo anterior, se pueden ejecutar simultáneamente un máximo de 10 funciones de orquestador o entidad. y 10 funciones de actividad en una única máquina virtual. Si no se especifica, el número de ejecuciones simultáneas de funciones de actividad y orquestador o entidad está limitado a 10 veces el número de núcleos de la máquina virtual.

Si se alcanza el número máximo de actividades, orquestaciones o entidades en una máquina virtual de trabajo, el desencadenador de Durable esperará a que las funciones en ejecución finalicen o se descarguen antes de iniciar nuevas ejecuciones de función.

> [!NOTE]
> Esta configuración resulta útil para ayudar a administrar el uso de memoria y de CPU en una única máquina virtual. Pero al escalar horizontalmente en varias máquinas virtuales, cada una tiene su propio conjunto de límites. Esta configuración no se puede usar para controlar la simultaneidad a nivel global.

> [!NOTE]
> Las orquestaciones y entidades solo se cargan en la memoria cuando están procesando eventos u operaciones de forma activa. Después de ejecutar su lógica y esperar (es decir, pulsar una instrucción `await` (C#) o `yield` (JavaScript, Python) en el código de función de orquestador), se descargan de la memoria. Las orquestaciones y entidades que se descargan de la memoria no cuentan para la limitación `maxConcurrentOrchestratorFunctions`. Incluso aunque haya millones de orquestaciones o entidades en el estado "En ejecución", no se tienen en cuenta para el límite a menos que se carguen en la memoria activa. Una orquestación que programa una función de actividad de forma similar no cuenta para la limitación si la orquestación está esperando a que la actividad termine de ejecutarse.

### <a name="language-runtime-considerations"></a>Consideraciones de entorno de ejecución de lenguaje

El entorno de ejecución de lenguaje que seleccione puede imponer restricciones de simultaneidad estrictas o sus funciones. Por ejemplo, es posible que las aplicaciones de Durable Function escritas en Python o PowerShell solo puedan admitir la ejecución de una sola función a la vez en una sola máquina virtual. Esto puede dar lugar a problemas de rendimiento significativos si no se tienen en cuenta. Por ejemplo, si un orquestador se expande a 10 actividades, pero el entorno de ejecución de lenguaje restringe la simultaneidad a una sola función, 9 de las 10 funciones de actividad quedarán bloqueadas a la espera de una oportunidad para ejecutarse. Además, estas 9 actividades bloqueadas no podrán equilibrar la carga en ningún otro trabajo porque el entorno de ejecución de Durable Functions ya los habrá cargado en la memoria. Esto se vuele especialmente problemático si las funciones de actividad son de ejecución prolongada.

Si el entorno de ejecución de lenguaje que el usuario utiliza implica una restricción en la simultaneidad, este debe actualizar la configuración de simultaneidad de Durable Functions para que coincida con la configuración de simultaneidad de su entorno de ejecución de lenguaje. Esto garantiza que el entorno de ejecución de Durable Functions no intentará ejecutar más funciones simultáneamente de las que permite el entorno de ejecución del lenguaje, de modo que las actividades pendientes equilibren la carga en otras máquinas virtuales. Por ejemplo, si el usuario tiene una aplicación de Python que restringe la simultaneidad a 4 funciones (quizás solo está configurada con 4 subprocesos en un único proceso de trabajo de lenguaje o 1 subproceso en 4 procesos de trabajo de lenguaje), debe establecer tanto `maxConcurrentOrchestratorFunctions` como `maxConcurrentActivityFunctions` en 4.

Para obtener más información y recomendaciones de rendimiento para Python, consulte [Mejora del rendimiento de las aplicaciones de Python en Azure Functions](../python-scale-performance-reference.md). Las técnicas mencionadas en esta documentación de referencia para desarrolladores de Python pueden tener un impacto considerable en el rendimiento y la escalabilidad de Durable Functions.

## <a name="extended-sessions"></a>Sesiones extendidas

Sesiones extendidas es un valor de configuración que mantiene las orquestaciones y entidades en memoria incluso después de que terminen de procesar mensajes. El efecto típico de habilitar sesiones extendidas es un número de operaciones de E/S reducido en lo que respecta al almacenamiento duradero subyacente y al rendimiento mejorado general.

Puede habilitar sesiones extendidas si establece `durableTask/extendedSessionsEnabled` en `true` en el archivo **host.json**. El valor `durableTask/extendedSessionIdleTimeoutInSeconds` se puede usar para controlar cuánto tiempo se retiene una sesión inactiva en memoria:

**Functions 2.0**
```json
{
  "extensions": {
    "durableTask": {
      "extendedSessionsEnabled": true,
      "extendedSessionIdleTimeoutInSeconds": 30
    }
  }
}
```

**Functions 1.0**
```json
{
  "durableTask": {
    "extendedSessionsEnabled": true,
    "extendedSessionIdleTimeoutInSeconds": 30
  }
}
```

Hay dos posibles inconvenientes de este valor que se deben tener en cuenta:

1. Hay un aumento general en el uso de memoria de la aplicación de funciones porque las instancias inactivas no se descargan de la memoria tan rápidamente.
2. Puede haber una disminución general del rendimiento si hay varias ejecuciones de funciones de orquestador o entidad simultáneas, distintas y de corta duración.

Por ejemplo, si `durableTask/extendedSessionIdleTimeoutInSeconds` se establece en 30 segundos, un episodio de función de orquestador o entidad de corta duración que se ejecuta en menos de 1 segundo sigue ocupando memoria durante 30 segundos. También se descuenta de la cuota `durableTask/maxConcurrentOrchestratorFunctions` mencionada antes, lo que puede evitar que se ejecuten otras funciones de orquestador o entidad.

En las secciones siguientes se describen los efectos concretos de las sesiones extendidas en las funciones de orquestador y entidad.

> [!NOTE]
> Las sesiones extendidas solo se admiten actualmente en lenguajes .NET, como C# o F#. Establecer `extendedSessionsEnabled` en `true` para otras plataformas puede conducir a problemas en tiempo de ejecución, como el error silencioso en la ejecución de las funciones desencadenadas por la actividad y la orquestación.

> [!NOTE]
> La compatibilidad con sesiones extendidas puede variar en función del [proveedor de almacenamiento de Durable Functions que use](durable-functions-storage-providers.md). Consulte la documentación del proveedor de almacenamiento para averiguar si admite sesiones extendidas.

### <a name="orchestrator-function-replay"></a>Reproducción de una función de orquestador

Tal como se ha mencionado anteriormente, las funciones de orquestador se reproducen con el contenido de la tabla **History**. De forma predeterminada, el código de la función de orquestador se reproduce cada vez que se elimina un lote de mensajes de una cola de control. Incluso si usa el patrón de distribución ramificada de entrada y salida y está esperando a que se completen todas las tareas (por ejemplo, con `Task.WhenAll()` en .NET, `context.df.Task.all()` en JavaScript, o `context.task_all()` en Python), habrá repeticiones que se produzcan a medida que se procesen lotes de respuestas de tareas a lo largo del tiempo. Cuando se habilitan sesiones extendidas, las instancias de funciones de orquestador se mantienen en memoria más tiempo y se pueden procesar mensajes nuevos sin una reproducción de historial completa.

La mejora del rendimiento de las sesiones extendidas suele notarse en las siguientes situaciones:

* Cuando hay un número limitado de instancias de orquestación que se ejecutan simultáneamente.
* Cuando las orquestaciones tienen un gran número de acciones secuenciales (por ejemplo, cientos de llamadas a funciones de actividad) que se completan rápidamente.
* Cuando las orquestaciones se ramifican en un gran número de acciones que se completan aproximadamente al mismo tiempo.
* Cuando las funciones de orquestador necesitan procesar mensajes grandes o realizar un procesamiento de datos con un uso intensivo de la CPU.

En todas las demás situaciones no suele notarse ninguna mejora del rendimiento para las funciones de orquestador.

> [!NOTE]
> Esta configuración solo se debe usar una vez que haya sido desarrollada y probada totalmente una función de orquestador. El comportamiento de reproducción agresivo predeterminado puede ser útil para detectar infracciones de [restricciones de código de función de orquestador](durable-functions-code-constraints.md) en tiempo de desarrollo y, por tanto, está deshabilitado de forma predeterminada.

### <a name="entity-function-unloading"></a>Descarga de funciones de entidad

Las funciones de entidad procesan hasta 20 operaciones en un único lote. En cuanto una entidad termina de procesar un lote de operaciones, conserva su estado y se descarga de la memoria. Puede retrasar la descarga de entidades de la memoria mediante la configuración de sesiones extendidas. Las entidades continúan conservando sus cambios de estado como antes, pero permanecen en memoria durante el período de tiempo configurado para reducir el número de cargas del almacenamiento. Esta reducción de cargas del almacenamiento puede mejorar el rendimiento general de las entidades a las que se accede con frecuencia.

## <a name="performance-targets"></a>Destinos del rendimiento

Si prevé utilizar Durable Functions para una aplicación de producción, es importante tener en cuenta los requisitos de rendimiento en una fase temprana del proceso de planeamiento. En esta sección se describen algunos escenarios de uso básico y las cifras de rendimiento máximo que se esperan.

* **Ejecución de actividad secuencial**: este escenario describe una función de orquestador que ejecuta una serie de funciones de actividad una tras otra. Se parece en gran medida al ejemplo de [encadenamiento de funciones](durable-functions-sequence.md).
* **Ejecución de actividad en paralelo**: este escenario describe una función de orquestador que ejecuta numerosas funciones de actividad en paralelo mediante el patrón de [distribución ramificada de entrada/salida](durable-functions-cloud-backup.md).
* **Procesamiento de respuestas en paralelo**: este escenario es la segunda mitad del patrón de [distribución ramificada de entrada/salida](durable-functions-cloud-backup.md). Se centra en el rendimiento de la distribución ramificada de entrada. Es importante tener en cuenta que, a diferencia de la distribución ramificada de salida, la distribución ramificada de entrada se realiza mediante una única instancia de la función de orquestador y, por lo tanto, solo se puede ejecutar en una sola máquina virtual.
* **Procesamiento de eventos externos**: este escenario representa una única instancia de la función de orquestador que espera en [eventos externos](durable-functions-external-events.md), uno a uno.
* **Procesamiento de operaciones de entidad**: En este escenario se prueba la rapidez con la que una _sola_[entidad Contador](durable-functions-entities.md) puede procesar un flujo constante de operaciones.

> [!TIP]
> A diferencia de la distribución ramificada de salida, las operaciones de la distribución ramificada de entrada están limitadas a una sola máquina virtual. Si la aplicación utiliza el patrón de distribución ramificada de entrada/salida, y le preocupa el rendimiento de la distribución ramificada de entrada, considere la posibilidad de subdividir la distribución ramificada de salida de la función de actividad entre varias [suborquestaciones](durable-functions-sub-orchestrations.md).

### <a name="azure-storage-performance-targets"></a>Objetivos de rendimiento de Azure Storage

En la tabla siguiente se muestran los números de rendimiento *máximo* esperados para los escenarios descritos anteriormente cuando se usa el [proveedor de Azure Storage predeterminado para Durable Functions](durable-functions-storage-providers.md#azure-storage). "Instancia" hace referencia a una sola instancia de una función de orquestador que se ejecuta en una única y pequeña máquina virtual ([A1](../../virtual-machines/sizes-previous-gen.md)) de Azure App Service. En todos los casos, se supone que las [sesiones extendidas](#orchestrator-function-replay) están habilitadas. Los resultados reales pueden variar según el trabajo de CPU o de E/S realizado por el código de la función.

| Escenario | Rendimiento máximo |
|-|-|
| Ejecución de actividad secuencial | 5 actividades por segundo y por instancia |
| Ejecución de actividad en paralelo (distribución ramificada de salida) | 100 actividades por segundo y por instancia |
| Procesamiento de respuestas en paralelo (distribución ramificada de entrada) | 150 respuestas por segundo y por instancia |
| Procesamiento de eventos externos | 50 eventos por segundo y por instancia |
| Procesamiento de operaciones de entidad | 64 operaciones por segundo |

Si no ve las cifras de rendimiento que espera, y el uso de CPU y de memoria parece correcto, compruebe si la causa está relacionada con el [mantenimiento de la cuenta de almacenamiento](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md#troubleshooting-guidance). La extensión de Durable Functions puede colocar una carga importante en una cuenta de Azure Storage y unas cargas suficientemente altas pueden provocar limitaciones en la cuenta de almacenamiento.

> [!TIP]
> En algunos casos, el usuario puede aumentar significativamente el rendimiento de los eventos externos, la actividad de distribución ramificada de entrada y las operaciones de entidad si aumenta el valor de la configuración `controlQueueBufferThreshold` en **host.jsen**. Aumentar este valor por encima de su valor predeterminado hace que el proveedor de almacenamiento del marco de tareas de Durable utilice más memoria para capturar previamente estos eventos de forma más agresiva, lo que reduce retrasos asociados con la eliminación de mensajes de las colas de control de Azure Storage. Para obtener más información, consulte la documentación de referencia de [host.json](durable-functions-bindings.md#host-json).

### <a name="high-throughput-processing"></a>Procesamiento de alto rendimiento

La arquitectura del back-end de Azure Storage pone ciertas limitaciones en la escalabilidad y el rendimiento teórico máximo de Durable Functions. Si las pruebas muestran que Durable Functions de Azure Storage no cumple sus requisitos de rendimiento, debe considerar el uso del [proveedor de almacenamiento Netherite para Durable Functions](durable-functions-storage-providers.md#netherite).

[Microsoft Research](https://www.microsoft.com/research) diseñó y desarrolló el back-end de almacenamiento de Netherite. Usa [Azure Event Hubs](../../event-hubs/event-hubs-about.md) y la tecnología de base de datos [FASTER](https://www.microsoft.com/research/project/faster/) además de [blobs en páginas de Azure](../../storage/blobs/storage-blob-pageblob-overview.md). El diseño de Netherite permite un procesamiento de orquestaciones y entidades significativamente más elevado en comparación con otros proveedores. En algunos escenarios de referencia, se ha demostrado que el rendimiento aumenta en más de un orden de magnitud en comparación con el proveedor de Azure Storage predeterminado.

Para más información sobre los proveedores de almacenamiento compatibles con Durable Functions y poder compararlos, consulte la documentación sobre [proveedores de almacenamiento de Durable Functions](durable-functions-storage-providers.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Información sobre la recuperación ante desastres y distribución geográfica](durable-functions-disaster-recovery-geo-distribution.md)
