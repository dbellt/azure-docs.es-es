---
title: 'Almacenamiento de tablas en columnas (versión preliminar) - Hiperescala (Citus): Azure Database for PostgreSQL'
description: Compresión de datos mediante el almacenamiento en columnas (versión preliminar)
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 04/07/2021
ms.openlocfilehash: c60c398d49a802dbe051ca37c4aea2092ab5144b
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107023908"
---
# <a name="columnar-table-storage-preview"></a>Almacenamiento de tablas en columnas (versión preliminar)

> [!IMPORTANT]
> El almacenamiento de tablas en columnas en Hiperescala (Citus) está actualmente en versión preliminar. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
>
> Puede ver la lista completa de características nuevas en [Características en versión preliminar de Hiperescala (Citus)](hyperscale-preview-features.md).

Azure Database for PostgreSQL: Hiperescala (Citus) admite el almacenamiento de tablas en columnas de solo anexar para cargas de trabajo de análisis y de almacenamiento de datos. Cuando las columnas (en lugar de las filas) se almacenan de forma contigua en el disco, los datos se comprimen más y las consultas pueden solicitar un subconjunto de columnas más rápidamente.

Para usar el almacenamiento en columnas, especifique `USING columnar` al crear una tabla:

```postgresql
CREATE TABLE contestant (
    handle TEXT,
    birthdate DATE,
    rating INT,
    percentile FLOAT,
    country CHAR(3),
    achievements TEXT[]
) USING columnar;
```

Hiperescala (Citus) convierte el almacenamiento en filas en almacenamiento en columnas en "franjas" durante la inserción. Cada franja contiene los datos de una transacción, o 150 000 filas, lo que sea menor.  (El tamaño de franja y otros parámetros de la tabla en columnas se pueden cambiar con la función [alter_columnar_table_set](reference-hyperscale-functions.md#alter_columnar_table_set) columna).

Por ejemplo, la instrucción siguiente coloca las cinco filas en la misma franja, ya que todos los valores se insertan en una única transacción:

```postgresql
-- insert these values into a single columnar stripe

INSERT INTO contestant VALUES
  ('a','1990-01-10',2090,97.1,'XA','{a}'),
  ('b','1990-11-01',2203,98.1,'XA','{a,b}'),
  ('c','1988-11-01',2907,99.4,'XB','{w,y}'),
  ('d','1985-05-05',2314,98.3,'XB','{}'),
  ('e','1995-05-05',2236,98.2,'XC','{a}');
```

Es mejor crear franjas grandes siempre que sea posible, ya que Hiperescala (Citus) comprime los datos en columnas por separado por franja. Podemos ver datos sobre nuestra tabla en columnas como la tasa de compresión, el número de franjas y el promedio de filas por franja mediante `VACUUM VERBOSE`:

```postgresql
VACUUM VERBOSE contestant;
```
```
INFO:  statistics for "contestant":
storage id: 10000000000
total file size: 24576, total data size: 248
compression rate: 1.31x
total row count: 5, stripe count: 1, average rows per stripe: 5
chunk count: 6, containing data for dropped columns: 0, zstd compressed: 6
```

La salida muestra que Hiperescala (Citus) usó el algoritmo de compresión zstd para obtener una compresión de datos de 1,31x. La tasa de compresión compara a) el tamaño de los datos insertados a medida que se agregaban a la memoria con b) el tamaño de los datos comprimidos en su franja final.

Por la manera en la que se mide, la tasa de compresión puede o no coincidir con la diferencia de tamaño entre el almacenamiento en filas y en columnas de una tabla. La única manera de encontrar realmente esa diferencia es crear una tabla de filas y columnas que contenga los mismos datos y comparar:

```postgresql
CREATE TABLE contestant_row AS
    SELECT * FROM contestant;

SELECT pg_total_relation_size('contestant_row') as row_size,
       pg_total_relation_size('contestant') as columnar_size;
```
```
 row_size | columnar_size
----------+---------------
    16384 |         24576
```

Para nuestra pequeña tabla, el almacenamiento en columnas realmente usa más espacio, pero a medida que los datos aumentan, la compresión ganará.

## <a name="example"></a>Ejemplo

El almacenamiento en columnas funciona bien con las particiones de tabla. Para un ejemplo, consulte la documentación de la comunidad del motor de Citus, [archivado con almacenamiento en columnas](https://docs.citusdata.com/en/stable/use_cases/timeseries.html#archiving-with-columnar-storage).

## <a name="gotchas"></a>Gotchas

* En el almacenamiento en columnas se comprime por franja. Las franjas se crean por transacción, por lo que la inserción de una fila por transacción colocará filas únicas en sus propias franjas. La compresión y el rendimiento de las franjas de fila única serán peores que los de una tabla de filas. La inserción debe realizarse siempre de manera masiva en las tablas en columnas.
* Si se desordena y se hace una columna de un grupo de franjas minúsculas, se produce un bloqueo.
  La única solución es crear una nueva tabla en columnas y copiar los datos del original en una transacción:
  ```postgresql
  BEGIN;
  CREATE TABLE foo_compacted (LIKE foo) USING columnar;
  INSERT INTO foo_compacted SELECT * FROM foo;
  DROP TABLE foo;
  ALTER TABLE foo_compacted RENAME TO foo;
  COMMIT;
  ```
* Fundamentalmente, los datos que no se pueden comprimir causan problemas, aunque el almacenamiento en columnas sigue siendo útil si se seleccionan columnas específicas. No es necesario cargar las demás columnas en la memoria.
* En una tabla con particiones con una combinación de particiones de filas y de columnas, las actualizaciones se deben dirigir cuidadosamente. Fíltrelas para ver solo las particiones de fila.
   * Si la operación está destinada a una partición de fila específica (por ejemplo, `UPDATE p2 SET i = i + 1`), se realizará correctamente; si se ha dirigido a una partición de columnas determinada (por ejemplo, `UPDATE p1 SET i = i + 1`), se producirá un error.
   * Si la operación está destinada a la tabla con particiones y tiene una cláusula WHERE que excluye todas las particiones de columnas (por ejemplo, `UPDATE parent SET i = i + 1 WHERE timestamp = '2020-03-15'`), se realizará correctamente.
   * Si la operación está destinada a la tabla con particiones, pero no filtra por las columnas de clave de la partición, se producirá un error. Aunque haya cláusulas WHERE que coincidan con las filas en las particiones solo de columnas, no es suficiente: la clave de partición también debe filtrarse.

## <a name="limitations"></a>Limitaciones

Esta característica todavía tiene una serie de limitaciones importantes. Consulte [Límite y limitaciones de Hiperescala (Citus)](concepts-hyperscale-limits.md#columnar-storage).

## <a name="next-steps"></a>Pasos siguientes

* Consulte un ejemplo de almacenamiento en columnas en un [tutorial de series temporales](https://docs.citusdata.com/en/stable/use_cases/timeseries.html#archiving-with-columnar-storage) de Citus (vínculo externo).
