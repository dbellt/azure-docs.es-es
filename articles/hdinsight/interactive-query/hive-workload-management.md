---
title: Característica Workload Management de Hive LLAP
titleSuffix: Azure HDInsight
description: Característica Workload Management de Hive LLAP
ms.service: hdinsight
ms.topic: how-to
author: guptanikhil007
ms.author: guptan
ms.reviewer: jasonh
ms.date: 05/25/2021
ms.openlocfilehash: 8dde65d0547552af90caddd214dd1ceda5a754ee
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110482926"
---
# <a name="hive-llap-workload-management-wlm-feature"></a>Característica Workload Management (WLM) de Hive LLAP
En un clúster de Interactive Query, la administración de recursos es imperativa, especialmente en un entorno multiinquilino. Hive LLAP (procesamiento analítico de baja latencia) usa la administración de cargas de trabajo para permitir que los usuarios busquen coincidencias con las necesidades específicas de la carga de trabajo y eviten la contención de esos recursos. <br> Workload Management implementa grupos de recursos (también conocidos como grupos de consultas), lo que permite dividir los recursos disponibles para Hive/LLAP en grupos que se usarán para cargas de trabajo específicas.
También permite configurar el porcentaje de recursos y el paralelismo de consultas para cada grupo de recursos individual.

![`Arquitectura de LLAP.`](./media/hive-workload-management/llap-architecture.png)

## <a name="enable-hive-llap-workload-management-feature-for-hdinsight-clusters"></a>Habilitación de la característica Workload Management de Hive LLAP para clústeres de HDInsight

Habilite la característica de administración de cargas de trabajo en los clústeres de Interactive Query de HDInsight siguiendo los pasos que se indican a continuación:
1. Cree una nueva cola de YARN, que se puede usar para abrir LOS AM de Tez de la administración de cargas de trabajo.
2. Cambie las configuraciones del clúster a través de Ambari para habilitar la característica en Hive.
3. Cree y active un plan de recursos.

### <a name="create-a-new-yarn-queue-suitable-for-workload-management-feature"></a>Creación de una nueva cola de YARN adecuada para la característica Workload Management
Cree una nueva cola de YARN denominada `wm` con la ayuda de la siguiente [guía](../hdinsight-troubleshoot-yarn.md).
Configure la cola `wm` en el clúster en función de las siguientes configuraciones:

| QueueName   | Capacity | Capacidad máxima | Priority | Recurso de AM máximo |
|------------|---------|--------------|----------|---------------------|
| `default`   | 5 %       | 5 %           | 0        | 33 %                 |
| `llap`      | 85%      | 100 %         | 10       | 33 %                 |
| `wm`        | 10 %      | 15 %          | 9        | 100 %                |

Confirme si la configuración de la cola `wm` tiene el aspecto que se muestra a continuación.
:::image type="content" source="./media/hive-workload-management/wm-yarn-queue.png" alt-text="Configuración de la cola wm.":::

### <a name="enable-workload-management-feature-in-hive-configs"></a>Habilitación de la característica Workload Management en configuraciones de Hive
Agregue la siguiente propiedad a Custom hiveserver2-interactive-site y establezca su valor en el nombre de la cola YARN recién creada; es decir, `wm`. Reinicie Interactive HiveServer para que surtan efecto los cambios de configuración.
```
hive.server2.tez.interactive.queue=wm
```

### <a name="create-resource-plan"></a>Creación del plan de recursos
A continuación se muestra un ejemplo sobre cómo crear un plan de recursos básico.
![`Plan de recursos básico.`](./media/hive-workload-management/wlm-resourceplan.jpg)

Ejecute los siguientes comandos a través de beeline para crear el plan de recursos anterior.

#### <a name="commands-to-create-view-and-drop-the-resource-plan"></a>Comandos para crear, ver y quitar el plan de recursos
```hql
# CREATE RESOURCE PLAN
CREATE RESOURCE PLAN demo_plan;

# CREATE POOLS
ALTER POOL demo_plan.default SET ALLOC_FRACTION = 0.65, QUERY_PARALLELISM = 2;
CREATE POOL demo_plan.etl WITH ALLOC_FRACTION = 0.20, QUERY_PARALLELISM = 2;
CREATE POOL demo_plan.sys_accounts WITH ALLOC_FRACTION = 0.15, QUERY_PARALLELISM = 1;

# CREATE MAPPING
CREATE USER MAPPING 'hive' IN demo_plan TO sys_accounts WITH ORDER 1;
 
# CREATE TRIGGERS
CREATE TRIGGER demo_plan.defaultToETL WHEN  ELAPSED_TIME > 20000 DO MOVE TO etl;
ALTER TRIGGER demo_plan.defaultToETL ADD TO POOL default;
CREATE TRIGGER demo_plan.ETLKill  WHEN ELAPSED_TIME > 40000 DO KILL;
ALTER TRIGGER demo_plan.ETLKill ADD TO POOL etl;

# VALIDATE PLAN
ALTER RESOURCE PLAN demo_plan VALIDATE;

# ENABLE PLAN
ALTER RESOURCE PLAN demo_plan ENABLE;

#  ACTIVATE PLAN
ALTER RESOURCE PLAN demo_plan ACTIVATE;

# SHOW RESOURCE PLAN
SHOW RESOURCE PLANS;
SHOW RESOURCE PLAN demo_plan;

# VALIDATE PLAN
ALTER RESOURCE PLAN demo_plan VALIDATE;

# ENABLE PLAN
ALTER RESOURCE PLAN demo_plan ENABLE;

#  ACTIVATE PLAN
ALTER RESOURCE PLAN demo_plan ACTIVATE;

# SHOW RESOURCE PLAN
SHOW RESOURCE PLANS;
SHOW RESOURCE PLAN demo_plan;

# DISABLE PLAN
-- In case plan is in active state first run:
-- DISABLE WORKLOAD MANAGEMENT;
ALTER RESOURCE PLAN demo_plan DISABLE;

# DROP RESOURCE PLAN
DROP RESOURCE PLAN demo_plan;
```

## <a name="understanding-resource-plan"></a>Descripción del plan de recursos
Para lograr un plan de recursos óptimo, es necesario tener un conocimiento profundo de los requisitos de la carga de trabajo.

### <a name="number-of-pools"></a>Número de grupos
El número de grupos está limitado por el paralelismo total de consultas (como mínimo, una consulta por grupo).
La mayoría de las cargas de trabajo rara vez requieren más de tres grupos. 
- predeterminado, para las consultas interactivas 
- etl/batch, para las consultas de larga duración
- sys, para los administradores del sistema

### <a name="total-query_parallelism"></a>Total de QUERY_PARALLELISM
El total de QUERY_PARALLELISM o el número total de consultas simultáneas se pueden obtener con la fórmula siguiente:

```
Number of total concurrent queries(Tez AMs) = Math.floor( (total cluster memory capacity / size of Tez AM container) x percentage of wm queue capacity)
```

Por ejemplo: <br/>
Supongamos que el tamaño del contenedor de AM de Tez es de 4 GB y que la capacidad total de memoria del clúster YARN es de 400 GB, de los cuales el 10 % se asigna para la cola wm. <br/>
Número total de consultas simultáneas = floor((400/4) x 0,10) = 10

> [!Tip]
> Se recomienda tener una capacidad ligeramente mayor de la necesaria en la cola wm para evitar que los AM de Tez se bloqueen en el estado aceptado; es decir, la capacidad de la cola `wm` se puede cambiar al 10,01 % y la capacidad de la cola `default` se puede reducir al 4,99 %.

### <a name="mappings"></a>Asignaciones
Las asignaciones proporcionan un mecanismo para dirigir las consultas a grupos determinados. A medida que aumenta el número de asignaciones, se pueden aplicar varias reglas para una consulta determinada. Para determinar qué regla debe tener prioridad, haga lo siguiente: si la ordenación está especificada mediante la cláusula `WITH ORDER` opcional, la regla con el menor valor de orden tiene prioridad. De lo contrario, las reglas `user` tienen prioridad sobre las reglas `application` y las reglas `application` tienen prioridad sobre las reglas `group`. <br/>
El orden de las reglas de grupo con la misma prioridad no está definido.


> [!Note]
> * Los AM de Tez en la cola `llap` permanecerán sin usar cuando el plan de WLM esté activo. Estos AM de Tez en la cola `llap` estarán disponibles fácilmente en caso de que se deshabilite el plan de recursos de WLM.
> * Al habilitar el plan de recursos de WLM, se inicia un número de AM de Tez igual al total de `QUERY_PARALLELISM` configurado para el plan de recursos especificado. El tamaño de la cola `wm` debe ajustarse para evitar que los AM de Tez se bloqueen en el estado ACCEPTED.
> * Solo se admite el uso de los dos contadores siguientes en los planes de recursos:
>    * EXECUTION_TIME
>    * ELAPSED_TIME

## <a name="related-articles"></a>Artículos relacionados
* [Resumen de comandos de Workload Management de Hive LLAP](workload-management-commands.md)
* [Solución de problemas de Workload Management de Hive LLAP](troubleshoot-workload-management-issues.md)


