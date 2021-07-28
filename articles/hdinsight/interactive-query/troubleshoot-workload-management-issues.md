---
title: Solución de problemas de Workload Management de Hive LLAP
titleSuffix: Azure HDInsight
description: Solución de problemas de Workload Management de Hive LLAP
ms.service: hdinsight
ms.topic: troubleshooting
author: guptanikhil007
ms.author: guptan
ms.reviewer: jasonh
ms.date: 05/25/2021
ms.openlocfilehash: 6a06a67b5039bc3a7e25e8300128c85ee008be3a
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110483139"
---
# <a name="troubleshoot-hive-llap-workload-management-issues"></a>Solución de problemas de Workload Management de Hive LLAP

Workload Management (WLM) está disponible para los clientes que inician clústeres de HDInsight 4.0. Use los recursos siguientes para ayudar a depurar problemas relacionados con la característica WLM.

## <a name="get-wlm-resource-plan-and-plan-entities"></a>Obtención de los planes de recursos de WLM y las entidades de plan
#### <a name="to-get-all-resource-plans-on-the-cluster"></a>Para obtener todos los planes de recursos del clúster:
```hql
SHOW RESOURCE PLANS;
```

#### <a name="to-get-definition-of-a-given-resource-plan"></a>Para obtener la definición de un plan de recursos determinado:
```hql
SHOW RESOURCE PLAN <plan_name>;
```

## <a name="get-wlm-entities-information-from-metastore-database"></a>Obtención de información de las entidades de WLM de la base de datos de metastore
> [!NOTE]
> Solo se aplica a la base de datos metastore de Hive personalizada

La información de las entidades de WLM también se puede ver en las tablas siguientes de la base de datos de metastore de Hive. 

* **WM_RESOURCEPLANS** (cadena NAME, cadena STATUS, número entero QUERY_PARALLELISM, cadena DEFAULT_POOL_PATH)
* **WM_POOLS** (cadena RP_NAME, cadena PATH, double ALLOC_FRACTION, número entero QUERY_PARALLELISM, cadena SCHEDULING_POLICY)
* **WM_MAPPINGS** (cadena RP_NAME, cadena ENTITY_TYPE, cadena ENTITY_NAME, cadena POOL_PATH, número entero ORDERING)
* **WM_TRIGGERS** (cadena RP_NAME, cadena NAME, cadena TRIGGER_EXPRESSION, cadena ACTION_EXPRESSION)
* **WM_POOLS_TO_TRIGGERS** (cadena RP_NAME, cadena POOL_PATH, cadena TRIGGER_NAME)

## <a name="wlm-metrics"></a>Métricas de WLM

Se puede acceder a las métricas de WLM directamente a través de la interfaz de usuario de HS2Interactive en la pestaña Metrics Dump (volcado de métricas). <br>
:::image type="content" source="./media/hive-workload-management/hs2-interactive-wlm.jpg" alt-text="Interfaz de usuario de HS2 Interactive." lightbox="./media/hive-workload-management/hs2-interactive-wlm.jpg":::

Métricas de ejemplo publicadas por WLM para un grupo determinado en un plan de recursos.
```
    "name" : "Hadoop:service=hiveserver2,name=WmPoolMetrics.etl",
    "modelerType" : "WmPoolMetrics.etl",
    "tag.Context" : "HS2",
    "tag.SessionId" : "etl",
    "tag.Hostname" : "hn0-c2b-ll.cu1cgjaim53urggr4psrgczloa.cx.internal.cloudapp.net",
    "NumExecutors" : 10,
    "NumRunningQueries" : 2,
    "NumParallelQueries" : 3,
    "NumQueuedQueries" : 0,
    "NumExecutorsMax" : 10
```

Es posible que la interfaz de usuario de HS2Interactive no funcione para los clústeres habilitados para ESP (Enterprise Security Package) publicados antes de abril de 2021. En tales casos, las métricas relacionadas con WLM se pueden obtener de paneles personalizados de Grafana.
<br>
El nombre de las métricas se rige por los patrones siguientes:
```
default.General.WM_<pool>_numExecutors
default.General.WM_<pool>_numExecutorsMax
default.General.WM_<pool>_numRunningQueries
default.General.WM_<pool>_numParallelQueries
default.General.WM_<pool>_numQueuedQueries
```
Reemplace `<pool>` por el nombre del grupo correspondiente para obtener las métricas en Grafana.

:::image type="content" source="./media/hive-workload-management/grafana-wlm.jpg" alt-text="Métricas de WLM en Grafana." lightbox="./media/hive-workload-management/grafana-wlm.jpg":::

> Nota: Asegúrese de que el componente hiveserver2 está seleccionado en los filtros anteriores y compruebe el nombre del componente.

<br>

## <a name="wlm-feature-characteristics"></a>Características de WLM
### <a name="lifecycle-of-tez-ams-in-wlm-enabled-clusters"></a>**Ciclo de vida de AM de Tez en clústeres habilitados para WLM**
A diferencia de los clústeres de LLAP predeterminados, los clústeres habilitados para WLM tienen otro conjunto de AM de Tez. Estos AM de Tez están programados para ejecutarse en la de cola `wm` si *hive.server2.tez.interactive.queue=wm* está configurado en los valores de Hive. <br>
Estos AM de Tez se generan cuando se activa WLM en función de la suma de QUERY_PARALLELISM de todos los grupos definidos en el plan de recursos. <br>
Cuando se deshabilita Workload Management en el clúster, estos AM de Tez se terminan automáticamente.
`{ DISABLE WORKLOAD MANAGEMENT; }`

### <a name="resource-contention"></a>**Contención de recursos**
En un clúster de LLAP habilitado para WLM, los recursos se comparten entre consultas en función de la configuración del plan de recursos. El uso compartido de recursos a veces hace lentas las consultas.
Se pueden realizar algunos ajustes en el plan de recursos para reducir la contención de recursos que se produce dentro de un grupo. Por ejemplo, `scheduling_policy` se puede definir como `fair`, que garantiza una distribución equitativa de recursos en el clúster para cada consulta asignada al grupo; o se puede definir como `fifo`, que garantiza todos los recursos para la primera consulta que llegue al grupo.<br>
En el ejemplo siguiente se muestra cómo establecer la directiva de programación para un grupo denominado `etl` en el plan de recursos `wlm_basic`:
```hql
ALTER POOL wlm_basic.etl SET SCHEDULING_POLICY = fair;
```
También se puede establecer la directiva de programación al crear el grupo:
```hql
CREATE POOL wlm_basic.default WITH ALLOC_FRACTION = 0.5, QUERY_PARALLELISM = 2, SCHEDULING_POLICY = fifo;
```

### <a name="query-failures-for-some-specific-use-cases"></a>**Errores de consulta para algunos casos de uso específicos**
La ejecución de consultas en WLM se puede terminar automáticamente en los siguientes casos:
1. Cuando se aplica el desencadenador de movimiento a una consulta y un grupo de destino que no tiene ningún AM de Tez disponible, la consulta se termina en su lugar. <br>
Lo descrito anteriormente es una limitación de diseño de la característica WLM. Como solución alternativa, puede aumentar la propiedad `QUERY_PARALLELISM` del grupo de destino para que, incluso en un escenario de carga máxima, las consultas enviadas al clúster puedan ser compatibles con este grupo. Además, ajuste el tamaño de la cola de `wm`para adaptarse a este cambio. <br>
2. Cuando WLM está deshabilitada, todas las consultas en proceso generarán un error con el siguiente patrón de excepción:
   ```
   FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.tez.TezTask. Dag received [DAG_TERMINATE, DAG_KILL] in RUNNING state.
   ```
3. Cuando un AM de Tez de WLM se termina manualmente, algunas de las consultas pueden generar un error con el siguiente patrón. <br/>Estas consultas deben ejecutarse sin problemas al volver a enviarse.
```
java.util.concurrent.CancellationException: Task was cancelled.
    at com.google.common.util.concurrent.AbstractFuture.cancellationExceptionWithCause(AbstractFuture.java:1349) ~[guava-28.0-jre.jar:?]
    at com.google.common.util.concurrent.AbstractFuture.getDoneValue(AbstractFuture.java:550) ~[guava-28.0-jre.jar:?]
    at com.google.common.util.concurrent.AbstractFuture.get(AbstractFuture.java:513) ~[guava-28.0-jre.jar:?]
    at com.google.common.util.concurrent.AbstractFuture$TrustedFuture.get(AbstractFuture.java:90) ~[guava-28.0-jre.jar:?]
    at com.google.common.util.concurrent.Uninterruptibles.getUninterruptibly(Uninterruptibles.java:237) ~[guava-28.0-jre.jar:?]
    at com.google.common.util.concurrent.Futures.getDone(Futures.java:1064) ~[guava-28.0-jre.jar:?]
    at com.google.common.util.concurrent.Futures$CallbackListener.run(Futures.java:1013) ~[guava-28.0-jre.jar:?]
    at com.google.common.util.concurrent.DirectExecutor.execute(DirectExecutor.java:30) ~[guava-28.0-jre.jar:?]
    at com.google.common.util.concurrent.AbstractFuture.executeListener(AbstractFuture.java:1137) ~[guava-28.0-jre.jar:?]
    at com.google.common.util.concurrent.AbstractFuture.complete(AbstractFuture.java:957) ~[guava-28.0-jre.jar:?]
    at com.google.common.util.concurrent.AbstractFuture.cancel(AbstractFuture.java:611) ~[guava-28.0-jre.jar:?]
    at com.google.common.util.concurrent.AbstractFuture$TrustedFuture.cancel(AbstractFuture.java:118) ~[guava-28.0-jre.jar:?]
    at org.apache.hadoop.hive.ql.exec.tez.WmTezSession$TimeoutRunnable.run(WmTezSession.java:264) ~[hive-exec-3.1.3.4.1.3.6.jar:3.1.3.4.1.3.6]
    at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511) [?:1.8.0_275]
    at java.util.concurrent.FutureTask.run(FutureTask.java:266) [?:1.8.0_275]
    at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180) ~[?:1.8.0_275]
    at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293) ~[?:1.8.0_275]
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [?:1.8.0_275]
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [?:1.8.0_275]
    at java.lang.Thread.run(Thread.java:748) [?:1.8.0_275]
```

## <a name="known-issues"></a>Problemas conocidos
* Los trabajos de Spark enviados a través de [Hive Warehouse Connector (HWC)](apache-hive-warehouse-connector.md) pueden experimentar errores intermitentes si el clúster de LLAP de destino tiene habilitada la característica WLM. <br>
  Para evitar los problemas anteriores, el cliente puede tener dos clústeres de LLAP, uno con WLM habilitado y otro sin WLM.
  A continuación, el cliente puede usar HWC para conectar su clúster de Spark al clúster de LLAP sin WLM.

* A veces, el comando `DISABLE WORKLOAD MANAGEMENT;` se bloquea durante mucho tiempo. <br>
Cancele el comando y compruebe el estado de los planes de recursos con el siguiente comando: `SHOW RESOURCE PLANS;`.
Compruebe si hay un plan de recursos activo disponible antes de volver a ejecutar el comando `DISABLE WORKLOAD MANAGEMENT`. <br>

* Algunos de los AM de Tez pueden seguir ejecutándose y no desaparecen con el comando `DISABLE WORKLOAD MANAGEMENT` o al reiniciar HS2. <br>
Termine estos AM de Tez mediante `yarn UI` o `yarn console application` después de deshabilitar Workload Management.

## <a name="related-articles"></a>Artículos relacionados
* [Workload Management de Hive LLAP](hive-workload-management.md)
* [Resumen de comandos de Workload Management de Hive LLAP](workload-management-commands.md)

