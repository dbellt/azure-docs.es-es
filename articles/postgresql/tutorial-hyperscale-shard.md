---
title: 'Tutorial: Datos de partición en nodos de trabajo en Azure Database for PostgreSQL: Hiperescala (Citus)'
description: En este tutorial se muestra cómo crear tablas distribuidas y visualizar la distribución de sus datos con Hiperescala (Citus) de Azure Database for PostgreSQL.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.custom: mvc
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 12/16/2020
ms.openlocfilehash: b0b076809801a4bb2f055d94ba481736b008b52c
ms.sourcegitcommit: 67cdbe905eb67e969d7d0e211d87bc174b9b8dc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111854889"
---
# <a name="tutorial-shard-data-on-worker-nodes-in-azure-database-for-postgresql--hyperscale-citus"></a>Tutorial: Datos de partición en nodos de trabajo en Azure Database for PostgreSQL: Hiperescala (Citus)

En este tutorial, usará Azure Database for PostgreSQL Hiperescala (Citus) para aprender lo siguiente:

> [!div class="checklist"]
> * Crear particiones distribuidas de hash.
> * Ver dónde se colocan las particiones de tabla.
> * Identificar una distribución sesgada.
> * Crear restricciones en tablas distribuidas.
> * Ejecutar consultas en datos distribuidos.

## <a name="prerequisites"></a>Prerrequisitos

En este tutorial se requiere un grupo de servidores de Hiperescala (Citus) en ejecución con dos nodos de trabajo. Si no tiene un grupo de servidores en ejecución, siga el tutorial de [creación de grupos de servidores](tutorial-hyperscale-server-group.md) y, después, vuelva a este.

## <a name="hash-distributed-data"></a>Distribución por hash

La distribución de las filas de la tabla entre varios servidores PostgreSQL es una técnica clave para la realización de consultas escalables en Hyperscale (Citus). Juntos, varios nodos pueden contener más datos que una base de datos tradicional y, en muchos casos, pueden usar CPU de trabajo en paralelo para ejecutar consultas.

En la sección de requisitos previos, creamos un grupo de servidores de Hiperescala (Citus) con dos nodos de trabajo.

![Coordinador y dos de trabajo](tutorial-hyperscale-shard/nodes.png)

Las tablas de metadatos del nodo de coordinador realizan el seguimiento de los trabajos y los datos distribuidos. Podemos consultar los trabajos activos en la tabla [pg_dist_node](reference-hyperscale-metadata.md#worker-node-table).

```sql
select nodeid, nodename from pg_dist_node where isactive;
```
```
 nodeid | nodename
--------+-----------
      1 | 10.0.0.21
      2 | 10.0.0.23
```

> [!NOTE]
> En Hiperescala (Citus), los nombres de nodos son direcciones IP internas de una red virtual y las direcciones reales que se ven puede que no sean las mismas.

### <a name="rows-shards-and-placements"></a>Filas, particiones y selecciones de ubicaciones

Para usar los recursos de almacenamiento y CPU de los nodos de trabajo, es preciso distribuir los datos de la tabla en todo el grupo de servidores.  Al distribuir una tabla se asigna cada fila a un grupo lógico denominado *partición.* Vamos a crear una tabla y a distribuirla:

```sql
-- create a table on the coordinator
create table users ( email text primary key, bday date not null );

-- distribute it into shards on workers
select create_distributed_table('users', 'email');
```

Hiperescala (Citus) asigna cada fila a una partición en función del valor de la *columna de distribución*, que, en nuestro caso, especificamos que sea `email`. Cada fila estará en una sola partición y cada partición puede contener varias filas.

![tabla de usuarios con filas que apuntan a particiones](tutorial-hyperscale-shard/table.png)

De forma predeterminada `create_distributed_table()` crea 32 particiones, tal como se puede ver contando la tabla de metadatos [pg_dist_shard](reference-hyperscale-metadata.md#shard-table):

```sql
select logicalrelid, count(shardid)
  from pg_dist_shard
 group by logicalrelid;
```
```
 logicalrelid | count
--------------+-------
 users        |    32
```

Hiperescala (Citus) usa la tabla `pg_dist_shard` para asignar filas a particiones, según un hash del valor de la columna de distribución. Los detalles del hash no son importantes para este tutorial. Lo que importa es que podemos realizar consultas para ver qué valores se asignan a cada identificador de partición:

```sql
-- Where would a row containing hi@test.com be stored?
-- (The value doesn't have to actually be present in users, the mapping
-- is a mathematical operation consulting pg_dist_shard.)
select get_shard_id_for_distribution_column('users', 'hi@test.com');
```
```
 get_shard_id_for_distribution_column
--------------------------------------
                               102008
```

La asignación de filas a particiones es puramente lógico. Se deben asignar particiones a nodos de trabajo concretos para el almacenamiento, en lo que Hiperescala (Citus) llama *selección de ubicación de compartición*.

![particiones asignadas a trabajos](tutorial-hyperscale-shard/shard-placement.png)

Las selecciones de ubicación de las particiones se pueden examinar en [pg_dist_placement](reference-hyperscale-metadata.md#shard-placement-table).
Su unión con las otras tablas de metadatos que hemos visto muestra el lugar en que reside cada partición.

```sql
-- limit the output to the first five placements

select
    shard.logicalrelid as table,
    placement.shardid as shard,
    node.nodename as host
from
    pg_dist_placement placement,
    pg_dist_node node,
    pg_dist_shard shard
where placement.groupid = node.groupid
  and shard.shardid = placement.shardid
order by shard
limit 5;
```
```
 table | shard  |    host
-------+--------+------------
 users | 102008 | 10.0.0.21
 users | 102009 | 10.0.0.23
 users | 102010 | 10.0.0.21
 users | 102011 | 10.0.0.23
 users | 102012 | 10.0.0.21
```

### <a name="data-skew"></a>Asimetría de datos

Los grupos de servidores funcionan más eficazmente cuando se colocan los datos de forma uniforme en todos los nodos de trabajo y cuando se colocan los datos relacionados juntos en los mismos trabajos. En esta sección nos centraremos en la primera parte, la uniformidad de la ubicación.

Para mostrarla, se van a crear datos de ejemplo para la tabla `users`:

```sql
-- load sample data
insert into users
select
    md5(random()::text) || '@test.com',
    date_trunc('day', now() - random()*'100 years'::interval)
from generate_series(1, 1000);
```

Para ver los tamaños de las particiones, se pueden ejecutar [funciones de tamaño de tabla](https://www.postgresql.org/docs/current/functions-admin.html#FUNCTIONS-ADMIN-DBSIZE) en las particiones.

```sql
-- sizes of the first five shards
select *
from
    run_command_on_shards('users', $cmd$
      select pg_size_pretty(pg_table_size('%1$s'));
    $cmd$)
order by shardid
limit 5;
```
```
 shardid | success | result
---------+---------+--------
  102008 | t       | 16 kB
  102009 | t       | 16 kB
  102010 | t       | 16 kB
  102011 | t       | 16 kB
  102012 | t       | 16 kB
```

Podemos ver que todas las particiones tienen el mismo tamaño. Ya se ha visto que las ubicaciones se distribuyen uniformemente entre los trabajos, por lo que podemos deducir que los nodos de trabajo contienen aproximadamente el mismo número de filas.

Las filas de nuestro ejemplo de `users` se distribuyen de forma uniforme debido a las propiedades de la columna de distribución, `email`.

1. El número de direcciones de correo electrónico era mayor o igual que el número de particiones.
2. El número de filas por dirección de correo electrónico era similar (en nuestro caso, exactamente una fila por dirección porque hemos declarado correo electrónico como una clave).

Si se elige alguna tabla y columna de distribución en la que se produzca un error en cualquiera de las propiedades terminará con un tamaño de datos desigual en los trabajos, es decir, se producirá un *sesgo en los datos*.

### <a name="add-constraints-to-distributed-data"></a>Adición de restricciones a los datos distribuidos

Hiperescala (Citus) permite seguir disfrutando de la seguridad de una base de datos relacional, incluidas las [restricciones de la base de datos](https://www.postgresql.org/docs/current/ddl-constraints.html),
aunque con una limitación: Dada la naturaleza de los sistemas distribuidos, Hiperescala (Citus) no hará referencia cruzada a las restricciones de unicidad ni a la integridad referencial entre los nodos de trabajo.

Veamos el ejemplo de tabla `users` con una tabla relacionada.

```sql
-- books that users own
create table books (
    owner_email text references users (email),
    isbn text not null,
    title text not null
);

-- distribute it
select create_distributed_table('books', 'owner_email');
```

Por motivos de eficacia, `books` se distribuye de la misma forma que `users`: por dirección de correo electrónico del propietario. La distribución de valores de columna similares se denomina [coubicación](concepts-hyperscale-colocation.md).

No tuvimos ningún problema al distribuir books con una clave externa a los usuarios, ya que la clave estaba en una columna de distribución. Sin embargo, aparecerían problemas para hacer que `isbn` fuera una clave:

```sql
-- will not work
alter table books add constraint books_isbn unique (isbn);
```
```
ERROR:  cannot create constraint on "books"
DETAIL: Distributed relations cannot have UNIQUE, EXCLUDE, or
        PRIMARY KEY constraints that do not include the partition column
        (with an equality operator if EXCLUDE).
```

En una tabla distribuida, lo mejor es que las columnas sean un módulo único de la columna de distribución:

```sql
-- a weaker constraint is allowed
alter table books add constraint books_isbn unique (owner_email, isbn);
```

La restricción anterior simplemente hace que el valor de isbn sea único por usuario. Otra opción consiste en que books sea una [tabla de referencia](howto-hyperscale-modify-distributed-tables.md#reference-tables), en lugar de una tabla distribuida, y crear una tabla distribuida independiente que asocie books con users.

## <a name="query-distributed-tables"></a>Consulta de tablas distribuidas

En las secciones anteriores, se ha visto la forma en que las filas de una tabla distribuida se colocan en particiones en los nodos de trabajo. La mayoría de las veces no es preciso saber cómo ni dónde se almacenan los datos en un grupo de servidores. Hiperescala (Citus) tiene un ejecutor de consultas distribuidas que divide automáticamente las consultas SQL normales. Los ejecuta en paralelo en los nodos de trabajo cercanos a los datos.

Por ejemplo, podemos ejecutar una consulta para buscar la edad media de los usuarios y tratar la tabla `users` distribuida como una tabla normal en el coordinador.

```sql
select avg(current_date - bday) as avg_days_old from users;
```
```
    avg_days_old
--------------------
 17926.348000000000
```

![Consulta que va a las particiones a través del coordinador](tutorial-hyperscale-shard/query-fragments.png)

En segundo plano, el ejecutor de Hiperescala (Citus) crea una consulta independiente para cada partición, ejecuta todas ellas en los trabajos y combina el resultado. Puede verlo si usa el comando EXPLAIN de PostgreSQL:

```sql
explain select avg(current_date - bday) from users;
```
```
                                  QUERY PLAN
----------------------------------------------------------------------------------
 Aggregate  (cost=500.00..500.02 rows=1 width=32)
   ->  Custom Scan (Citus Adaptive)  (cost=0.00..0.00 rows=100000 width=16)
     Task Count: 32
     Tasks Shown: One of 32
     ->  Task
       Node: host=10.0.0.21 port=5432 dbname=citus
       ->  Aggregate  (cost=41.75..41.76 rows=1 width=16)
         ->  Seq Scan on users_102040 users  (cost=0.00..22.70 rows=1270 width=4)
```

La salida muestra un ejemplo de un plan de ejecución para un *fragmento de la consulta* que se ejecuta en la partición 102040 (la tabla `users_102040` en el host 10.0.0.21 de trabajo). Los restantes fragmentos no se muestran porque son similares. Se puede ver que el nodo de trabajo examina las tablas de particiones y aplica el agregado. El nodo de coordinación combina los agregados para obtener resultado final.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado una tabla distribuida y ha obtenido información sobre sus particiones y ubicaciones. Se ha visto el desafío de usar las restricciones de unicidad y una clave externa y, por último, se ha visto el funcionamiento de las consultas distribuidas en un nivel alto.

* Obtenga más información sobre los [tipos de tabla](concepts-hyperscale-nodes.md) de Hiperescala (Citus)
* Obtenga más sugerencias sobre [cómo elegir una columna de distribución](concepts-hyperscale-choose-distribution-column.md)
* Obtenga información sobre las ventajas de la [coubicación de tablas](concepts-hyperscale-colocation.md)
