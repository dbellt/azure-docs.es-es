---
title: 'Almacenamiento de tablas en columnas (versión preliminar) - Hiperescala (Citus): Azure Database for PostgreSQL'
description: Compresión de datos mediante el almacenamiento en columnas (versión preliminar)
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 05/04/2021
ms.openlocfilehash: c42cfcc35edf33cf30c4d69b4a1fb15d39dd4009
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108773690"
---
# <a name="columnar-table-storage-preview"></a>Almacenamiento de tablas en columnas (versión preliminar)

> [!IMPORTANT]
> El almacenamiento de tablas en columnas en Hiperescala (Citus) está actualmente en versión preliminar. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
>
> Puede ver la lista completa de características nuevas en [Características en versión preliminar de Hiperescala (Citus)](hyperscale-preview-features.md).

Azure Database for PostgreSQL: Hiperescala (Citus) admite el almacenamiento de tablas en columnas de solo anexar para cargas de trabajo de análisis y de almacenamiento de datos. Cuando las columnas (en lugar de las filas) se almacenan de forma contigua en el disco, los datos se comprimen más y las consultas pueden solicitar un subconjunto de columnas más rápidamente.

## <a name="usage"></a>Uso

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

Por la manera en la que se mide, la tasa de compresión puede o no coincidir con la diferencia de tamaño entre el almacenamiento en filas y en columnas de una tabla. La única manera de encontrar realmente esa diferencia es crear una tabla de filas y columnas que contenga los mismos datos y comparar.

## <a name="measuring-compression"></a>Medición de la compresión

Vamos a crear un nuevo ejemplo con más datos para comparar el ahorro de compresión.

```postgresql
-- first a wide table using row storage
CREATE TABLE perf_row(
  c00 int8, c01 int8, c02 int8, c03 int8, c04 int8, c05 int8, c06 int8, c07 int8, c08 int8, c09 int8,
  c10 int8, c11 int8, c12 int8, c13 int8, c14 int8, c15 int8, c16 int8, c17 int8, c18 int8, c19 int8,
  c20 int8, c21 int8, c22 int8, c23 int8, c24 int8, c25 int8, c26 int8, c27 int8, c28 int8, c29 int8,
  c30 int8, c31 int8, c32 int8, c33 int8, c34 int8, c35 int8, c36 int8, c37 int8, c38 int8, c39 int8,
  c40 int8, c41 int8, c42 int8, c43 int8, c44 int8, c45 int8, c46 int8, c47 int8, c48 int8, c49 int8,
  c50 int8, c51 int8, c52 int8, c53 int8, c54 int8, c55 int8, c56 int8, c57 int8, c58 int8, c59 int8,
  c60 int8, c61 int8, c62 int8, c63 int8, c64 int8, c65 int8, c66 int8, c67 int8, c68 int8, c69 int8,
  c70 int8, c71 int8, c72 int8, c73 int8, c74 int8, c75 int8, c76 int8, c77 int8, c78 int8, c79 int8,
  c80 int8, c81 int8, c82 int8, c83 int8, c84 int8, c85 int8, c86 int8, c87 int8, c88 int8, c89 int8,
  c90 int8, c91 int8, c92 int8, c93 int8, c94 int8, c95 int8, c96 int8, c97 int8, c98 int8, c99 int8
);

-- next a table with identical columns using columnar storage
CREATE TABLE perf_columnar(LIKE perf_row) USING COLUMNAR;
```

Rellene ambas tablas con el mismo conjunto de datos grande:

```postgresql
INSERT INTO perf_row
  SELECT
    g % 00500, g % 01000, g % 01500, g % 02000, g % 02500, g % 03000, g % 03500, g % 04000, g % 04500, g % 05000,
    g % 05500, g % 06000, g % 06500, g % 07000, g % 07500, g % 08000, g % 08500, g % 09000, g % 09500, g % 10000,
    g % 10500, g % 11000, g % 11500, g % 12000, g % 12500, g % 13000, g % 13500, g % 14000, g % 14500, g % 15000,
    g % 15500, g % 16000, g % 16500, g % 17000, g % 17500, g % 18000, g % 18500, g % 19000, g % 19500, g % 20000,
    g % 20500, g % 21000, g % 21500, g % 22000, g % 22500, g % 23000, g % 23500, g % 24000, g % 24500, g % 25000,
    g % 25500, g % 26000, g % 26500, g % 27000, g % 27500, g % 28000, g % 28500, g % 29000, g % 29500, g % 30000,
    g % 30500, g % 31000, g % 31500, g % 32000, g % 32500, g % 33000, g % 33500, g % 34000, g % 34500, g % 35000,
    g % 35500, g % 36000, g % 36500, g % 37000, g % 37500, g % 38000, g % 38500, g % 39000, g % 39500, g % 40000,
    g % 40500, g % 41000, g % 41500, g % 42000, g % 42500, g % 43000, g % 43500, g % 44000, g % 44500, g % 45000,
    g % 45500, g % 46000, g % 46500, g % 47000, g % 47500, g % 48000, g % 48500, g % 49000, g % 49500, g % 50000
  FROM generate_series(1,50000000) g;

INSERT INTO perf_columnar
  SELECT
    g % 00500, g % 01000, g % 01500, g % 02000, g % 02500, g % 03000, g % 03500, g % 04000, g % 04500, g % 05000,
    g % 05500, g % 06000, g % 06500, g % 07000, g % 07500, g % 08000, g % 08500, g % 09000, g % 09500, g % 10000,
    g % 10500, g % 11000, g % 11500, g % 12000, g % 12500, g % 13000, g % 13500, g % 14000, g % 14500, g % 15000,
    g % 15500, g % 16000, g % 16500, g % 17000, g % 17500, g % 18000, g % 18500, g % 19000, g % 19500, g % 20000,
    g % 20500, g % 21000, g % 21500, g % 22000, g % 22500, g % 23000, g % 23500, g % 24000, g % 24500, g % 25000,
    g % 25500, g % 26000, g % 26500, g % 27000, g % 27500, g % 28000, g % 28500, g % 29000, g % 29500, g % 30000,
    g % 30500, g % 31000, g % 31500, g % 32000, g % 32500, g % 33000, g % 33500, g % 34000, g % 34500, g % 35000,
    g % 35500, g % 36000, g % 36500, g % 37000, g % 37500, g % 38000, g % 38500, g % 39000, g % 39500, g % 40000,
    g % 40500, g % 41000, g % 41500, g % 42000, g % 42500, g % 43000, g % 43500, g % 44000, g % 44500, g % 45000,
    g % 45500, g % 46000, g % 46500, g % 47000, g % 47500, g % 48000, g % 48500, g % 49000, g % 49500, g % 50000
  FROM generate_series(1,50000000) g;

VACUUM (FREEZE, ANALYZE) perf_row;
VACUUM (FREEZE, ANALYZE) perf_columnar;
```

Para estos datos, puede ver una proporción de compresión superior a 8X en la tabla de columnas.

```postgresql
SELECT pg_total_relation_size('perf_row')::numeric/
       pg_total_relation_size('perf_columnar') AS compression_ratio;
 compression_ratio
--------------------
 8.0196135873627944
(1 row)
```

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

Esta característica todavía tiene limitaciones importantes. Consulte [Límite y limitaciones de Hiperescala (Citus)](concepts-hyperscale-limits.md#columnar-storage).

## <a name="next-steps"></a>Pasos siguientes

* Consulte un ejemplo de almacenamiento en columnas en un [tutorial de series temporales](https://docs.citusdata.com/en/stable/use_cases/timeseries.html#archiving-with-columnar-storage) de Citus (vínculo externo).
