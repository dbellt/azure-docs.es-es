---
title: 'Extensiones en Azure Database for PostgreSQL: servidor único'
description: 'Obtenga información sobre las extensiones de Postgres disponibles en Azure Database for PostgreSQL: servidor único'
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.topic: conceptual
ms.date: 03/25/2021
ms.openlocfilehash: 765c3653dae4f514263cd8db636d9cec42d56d15
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107107625"
---
# <a name="postgresql-extensions-in-azure-database-for-postgresql---single-server"></a>Extensiones de PostgreSQL en Azure Database for PostgreSQL: un solo servidor
PostgreSQL ofrece la capacidad de ampliar la funcionalidad de su base de datos mediante extensiones. Las extensiones agrupan varios objetos SQL relacionados en un solo paquete que se puede cargar o quitar de la base de datos con un solo comando. Después de cargarse en la base de datos, las extensiones funcionan como características integradas.

## <a name="how-to-use-postgresql-extensions"></a>¿Cómo se utilizan las extensiones de PostgreSQL?
Para poder usar las extensiones de PostgreSQL, es preciso que estén instaladas en su base de datos. Para instalar una extensión determinada, ejecute el comando [CREATE EXTENSION](https://www.postgresql.org/docs/current/sql-createextension.html) desde la herramienta psql para cargar los objetos empaquetados en la base de datos.

Azure Database for PostgreSQL admite un subconjunto de extensiones de claves, como se enumeran a continuación. Esta información también está disponible al ejecutar `SELECT * FROM pg_available_extensions;`. Aparte de las que se indican, no se admiten otras extensiones. -No puede crear su propia extensión en el servicio Azure Database for PostgreSQL.

## <a name="postgres-11-extensions"></a>Extensiones de Postgres 11

Las extensiones siguientes están disponibles en los servidores de Azure Database for PostgreSQL que tienen la versión 11 de Postgres. 

> [!div class="mx-tableFixed"]
> | **Extensión**| **Versión de la extensión** | **Descripción** |
> |---|---|---|
> |[address_standardizer](http://postgis.net/docs/Address_Standardizer.html)         | 2.5.1           | Se utilizan para analizar una dirección en los elementos que la componen. |
> |[address_standardizer_data_us](http://postgis.net/docs/Address_Standardizer.html) | 2.5.1           | Aborda el ejemplo del conjunto de datos estandarizado de EE. UU.|
> |[btree_gin](https://www.postgresql.org/docs/11/btree-gin.html)                    | 1.3             | Compatibilidad con la indexación de tipos de datos comunes en GIN|
> |[btree_gist](https://www.postgresql.org/docs/11/btree-gist.html)                   | 1.5             | Compatibilidad con la indexación de tipos de datos comunes en GiST|
> |[citext](https://www.postgresql.org/docs/11/citext.html)                       | 1.5             | Tipo de datos para cadenas de caracteres que no distinguen mayúsculas de minúsculas|
> |[cube](https://www.postgresql.org/docs/11/cube.html)                         | 1.4             | Tipo de datos para los cubos multidimensionales|
> |[dblink](https://www.postgresql.org/docs/11/dblink.html)                       | 1.2             | Se conecta a otras bases de datos de PostgreSQL desde una base de datos|
> |[dict_int](https://www.postgresql.org/docs/11/dict-int.html)                     | 1.0             | Plantilla de diccionario de búsqueda de texto para números enteros|
> |[earthdistance](https://www.postgresql.org/docs/11/earthdistance.html)                | 1.1             | Calcula distancias de círculo máximo en la superficie de la Tierra|
> |[fuzzystrmatch](https://www.postgresql.org/docs/11/fuzzystrmatch.html)                | 1.1             | Determina las similitudes y la distancia entre las cadenas|
> |[hstore](https://www.postgresql.org/docs/11/hstore.html)                       | 1.5             | Tipo de datos para almacenar conjuntos de pares (clave/valor)|
> |[hypopg](https://hypopg.readthedocs.io/en/latest/)                       | 1.1.2           | Índices hipotéticos para PostgreSQL|
> |[intarray](https://www.postgresql.org/docs/11/intarray.html)                     | 1.2             | Funciones, operadores e índices compatibles con matrices 1D de números enteros|
> |[isn](https://www.postgresql.org/docs/11/isn.html)                          | 1.2             | Tipos de datos para los estándares internacionales de numeración de productos|
> |[ltree](https://www.postgresql.org/docs/11/ltree.html)                        | 1.1             | Tipo de datos para las estructuras de árbol jerárquicas|
> |[orafce](https://github.com/orafce/orafce)                       | 3.7             | Funciones y operadores que emulan un subconjunto de funciones y paquetes desde un RDBMS comercial|
> |[pgaudit](https://www.pgaudit.org/)                     | 1.3.1             | Proporciona funcionalidad de auditoría|
> |[pgcrypto](https://www.postgresql.org/docs/11/pgcrypto.html)                     | 1.3             | Funciones de cifrado|
> |[pgrouting](https://pgrouting.org/)                    | 2.6.2           | Extensión pgRouting|
> |[pgrowlocks](https://www.postgresql.org/docs/11/pgrowlocks.html)                   | 1.2             | Muestra información de bloqueo de nivel de fila|
> |[pgstattuple](https://www.postgresql.org/docs/11/pgstattuple.html)                  | 1.5             | Muestra estadísticas de nivel de tupla|
> |[pg_buffercache](https://www.postgresql.org/docs/11/pgbuffercache.html)               | 1.3             | Examina la caché del búfer compartido|
> |[pg_partman](https://github.com/pgpartman/pg_partman)                   | 4.0.0           | Extensión para administrar tablas con particiones por hora o identificador|
> |[pg_prewarm](https://www.postgresql.org/docs/11/pgprewarm.html)                   | 1.2             | Prepara los datos de relación|
> |[pg_stat_statements](https://www.postgresql.org/docs/11/pgstatstatements.html)           | 1.6             | Realiza un seguimiento de las estadísticas de ejecución de todas las instrucciones SQL ejecutadas|
> |[pg_trgm](https://www.postgresql.org/docs/11/pgtrgm.html)                      | 1.4             | Medición de similitud de texto y búsqueda de índice basada en trigramas|
> |[plpgsql](https://www.postgresql.org/docs/11/plpgsql.html)                      | 1.0             | Lenguaje de procedimientos de PL/pgSQL|
> |[plv8](https://plv8.github.io/)                         | 2.3.11          | Lenguaje de procedimientos de confianza de PL/JavaScript (v8)|
> |[postgis](https://www.postgis.net/)                      | 2.5.1           | Geometría, geografía y funciones y tipos espaciales de trama de PostGIS|
> |[postgis_sfcgal](https://www.postgis.net/)               | 2.5.1           | Funciones de PostGIS SFCGAL|
> |[postgis_tiger_geocoder](https://www.postgis.net/)       | 2.5.1           | Geocoder de PostGIS tiger y geocoder inverso|
> |[postgis_topology](https://postgis.net/docs/Topology.html)             | 2.5.1           | Funciones y tipos espaciales de topología PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/11/postgres-fdw.html)                 | 1.0             | Contenedor de datos externos para servidores PostgreSQL remotos|
> |[tablefunc](https://www.postgresql.org/docs/11/tablefunc.html)                    | 1.0             | Funciones que manipulan la totalidad del contenido de las tablas, incluidas tablas de referencias cruzadas|
> |[timescaledb](https://docs.timescale.com/latest)                    |1.7.4             | Habilita las inserciones escalables y las consultas complejas para los datos de serie temporal|
> |[unaccent](https://www.postgresql.org/docs/11/unaccent.html)                     | 1.1             | Diccionario de búsqueda de texto que quita los acentos|
> |[uuid-ossp](https://www.postgresql.org/docs/11/uuid-ossp.html)                    | 1.1             | Genera identificadores únicos universales (UUID)|

## <a name="postgres-10-extensions"></a>Extensiones de Postgres 10 

Las extensiones siguientes están disponibles en los servidores de Azure Database for PostgreSQL que tienen la versión 10 de Postgres.

> [!div class="mx-tableFixed"]
> | **Extensión**| **Versión de la extensión** | **Descripción** |
> |---|---|---|
> |[address_standardizer](http://postgis.net/docs/Address_Standardizer.html)         | 2.5.1           | Se utilizan para analizar una dirección en los elementos que la componen. |
> |[address_standardizer_data_us](http://postgis.net/docs/Address_Standardizer.html) | 2.5.1           | Aborda el ejemplo del conjunto de datos estandarizado de EE. UU.|
> |[btree_gin](https://www.postgresql.org/docs/10/btree-gin.html)                    | 1.3             | Compatibilidad con la indexación de tipos de datos comunes en GIN|
> |[btree_gist](https://www.postgresql.org/docs/10/btree-gist.html)                   | 1.5             | Compatibilidad con la indexación de tipos de datos comunes en GiST|
> |[chkpass](https://www.postgresql.org/docs/10/chkpass.html)                       | 1.0             | Tipo de datos para las contraseñas de cifrado automático|
> |[citext](https://www.postgresql.org/docs/10/citext.html)                       | 1.4             | Tipo de datos para cadenas de caracteres que no distinguen mayúsculas de minúsculas|
> |[cube](https://www.postgresql.org/docs/10/cube.html)                         | 1.2             | Tipo de datos para los cubos multidimensionales|
> |[dblink](https://www.postgresql.org/docs/10/dblink.html)                       | 1.2             | Se conecta a otras bases de datos de PostgreSQL desde una base de datos|
> |[dict_int](https://www.postgresql.org/docs/10/dict-int.html)                     | 1.0             | Plantilla de diccionario de búsqueda de texto para números enteros|
> |[earthdistance](https://www.postgresql.org/docs/10/earthdistance.html)                | 1.1             | Calcula distancias de círculo máximo en la superficie de la Tierra|
> |[fuzzystrmatch](https://www.postgresql.org/docs/10/fuzzystrmatch.html)                | 1.1             | Determina las similitudes y la distancia entre las cadenas|
> |[hstore](https://www.postgresql.org/docs/10/hstore.html)                       | 1.4             | Tipo de datos para almacenar conjuntos de pares (clave/valor)|
> |[hypopg](https://hypopg.readthedocs.io/en/latest/)                       | 1.1.1           | Índices hipotéticos para PostgreSQL|
> |[intarray](https://www.postgresql.org/docs/10/intarray.html)                     | 1.2             | Funciones, operadores e índices compatibles con matrices 1D de números enteros|
> |[isn](https://www.postgresql.org/docs/10/isn.html)                          | 1.1             | Tipos de datos para los estándares internacionales de numeración de productos|
> |[ltree](https://www.postgresql.org/docs/10/ltree.html)                        | 1.1             | Tipo de datos para las estructuras de árbol jerárquicas|
> |[orafce](https://github.com/orafce/orafce)                       | 3.7             | Funciones y operadores que emulan un subconjunto de funciones y paquetes desde un RDBMS comercial|
> |[pgaudit](https://www.pgaudit.org/)                     | 1.2             | Proporciona funcionalidad de auditoría|
> |[pgcrypto](https://www.postgresql.org/docs/10/pgcrypto.html)                     | 1.3             | Funciones de cifrado|
> |[pgrouting](https://pgrouting.org/)                    | 2.5.2           | Extensión pgRouting|
> |[pgrowlocks](https://www.postgresql.org/docs/10/pgrowlocks.html)                   | 1.2             | Muestra información de bloqueo de nivel de fila|
> |[pgstattuple](https://www.postgresql.org/docs/10/pgstattuple.html)                  | 1.5             | Muestra estadísticas de nivel de tupla|
> |[pg_buffercache](https://www.postgresql.org/docs/10/pgbuffercache.html)               | 1.3             | Examina la caché del búfer compartido|
> |[pg_partman](https://github.com/pgpartman/pg_partman)                   | 2.6.3           | Extensión para administrar tablas con particiones por hora o identificador|
> |[pg_prewarm](https://www.postgresql.org/docs/10/pgprewarm.html)                   | 1.1             | Prepara los datos de relación|
> |[pg_stat_statements](https://www.postgresql.org/docs/10/pgstatstatements.html)           | 1.6             | Realiza un seguimiento de las estadísticas de ejecución de todas las instrucciones SQL ejecutadas|
> |[pg_trgm](https://www.postgresql.org/docs/10/pgtrgm.html)                      | 1.3             | Medición de similitud de texto y búsqueda de índice basada en trigramas|
> |[plpgsql](https://www.postgresql.org/docs/10/plpgsql.html)                      | 1.0             | Lenguaje de procedimientos de PL/pgSQL|
> |[plv8](https://plv8.github.io/)                         | 2.1.0          | Lenguaje de procedimientos de confianza de PL/JavaScript (v8)|
> |[postgis](https://www.postgis.net/)                      | 2.4.3           | Geometría, geografía y funciones y tipos espaciales de trama de PostGIS|
> |[postgis_sfcgal](https://www.postgis.net/)               | 2.4.3           | Funciones de PostGIS SFCGAL|
> |[postgis_tiger_geocoder](https://www.postgis.net/)       | 2.4.3           | Geocoder de PostGIS tiger y geocoder inverso|
> |[postgis_topology](https://postgis.net/docs/Topology.html)             | 2.4.3           | Funciones y tipos espaciales de topología PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/10/postgres-fdw.html)                 | 1.0             | Contenedor de datos externos para servidores PostgreSQL remotos|
> |[tablefunc](https://www.postgresql.org/docs/10/tablefunc.html)                    | 1.0             | Funciones que manipulan la totalidad del contenido de las tablas, incluidas tablas de referencias cruzadas|
> |[timescaledb](https://docs.timescale.com/latest)                    | 1.7.4             | Habilita las inserciones escalables y las consultas complejas para los datos de serie temporal|
> |[unaccent](https://www.postgresql.org/docs/10/unaccent.html)                     | 1.1             | Diccionario de búsqueda de texto que quita los acentos|
> |[uuid-ossp](https://www.postgresql.org/docs/10/uuid-ossp.html)                    | 1.1             | Genera identificadores únicos universales (UUID)|

## <a name="postgres-96-extensions"></a>Extensiones de Postgres 9.6 

Las extensiones siguientes están disponibles en los servidores de Azure Database for PostgreSQL que tienen la versión 9.6 de Postgres.

> [!div class="mx-tableFixed"]
> | **Extensión**| **Versión de la extensión** | **Descripción** |
> |---|---|---|
> |[address_standardizer](http://postgis.net/docs/Address_Standardizer.html)         | 2.3.2           | Se utilizan para analizar una dirección en los elementos que la componen. |
> |[address_standardizer_data_us](http://postgis.net/docs/Address_Standardizer.html) | 2.3.2           | Aborda el ejemplo del conjunto de datos estandarizado de EE. UU.|
> |[btree_gin](https://www.postgresql.org/docs/9.6/btree-gin.html)                    | 1.0             | Compatibilidad con la indexación de tipos de datos comunes en GIN|
> |[btree_gist](https://www.postgresql.org/docs/9.6/btree-gist.html)                   | 1.2             | Compatibilidad con la indexación de tipos de datos comunes en GiST|
> |[chkpass](https://www.postgresql.org/docs/9.6/chkpass.html)                       | 1.0             | Tipo de datos para las contraseñas de cifrado automático|
> |[citext](https://www.postgresql.org/docs/9.6/citext.html)                       | 1.3             | Tipo de datos para cadenas de caracteres que no distinguen mayúsculas de minúsculas|
> |[cube](https://www.postgresql.org/docs/9.6/cube.html)                         | 1.2             | Tipo de datos para los cubos multidimensionales|
> |[dblink](https://www.postgresql.org/docs/9.6/dblink.html)                       | 1.2             | Se conecta a otras bases de datos de PostgreSQL desde una base de datos|
> |[dict_int](https://www.postgresql.org/docs/9.6/dict-int.html)                     | 1.0             | Plantilla de diccionario de búsqueda de texto para números enteros|
> |[earthdistance](https://www.postgresql.org/docs/9.6/earthdistance.html)                | 1.1             | Calcula distancias de círculo máximo en la superficie de la Tierra|
> |[fuzzystrmatch](https://www.postgresql.org/docs/9.6/fuzzystrmatch.html)                | 1.1             | Determina las similitudes y la distancia entre las cadenas|
> |[hstore](https://www.postgresql.org/docs/9.6/hstore.html)                       | 1.4             | Tipo de datos para almacenar conjuntos de pares (clave/valor)|
> |[hypopg](https://hypopg.readthedocs.io/en/latest/)                       | 1.1.1           | Índices hipotéticos para PostgreSQL|
> |[intarray](https://www.postgresql.org/docs/9.6/intarray.html)                     | 1.2             | Funciones, operadores e índices compatibles con matrices 1D de números enteros|
> |[isn](https://www.postgresql.org/docs/9.6/isn.html)                          | 1.1             | Tipos de datos para los estándares internacionales de numeración de productos|
> |[ltree](https://www.postgresql.org/docs/9.6/ltree.html)                        | 1.1             | Tipo de datos para las estructuras de árbol jerárquicas|
> |[orafce](https://github.com/orafce/orafce)                       | 3.7             | Funciones y operadores que emulan un subconjunto de funciones y paquetes desde un RDBMS comercial|
> |[pgaudit](https://www.pgaudit.org/)                     | 1.1.2             | Proporciona funcionalidad de auditoría|
> |[pgcrypto](https://www.postgresql.org/docs/9.6/pgcrypto.html)                     | 1.3             | Funciones de cifrado|
> |[pgrouting](https://pgrouting.org/)                    | 2.3.2           | Extensión pgRouting|
> |[pgrowlocks](https://www.postgresql.org/docs/9.6/pgrowlocks.html)                   | 1.2             | Muestra información de bloqueo de nivel de fila|
> |[pgstattuple](https://www.postgresql.org/docs/9.6/pgstattuple.html)                  | 1.4             | Muestra estadísticas de nivel de tupla|
> |[pg_buffercache](https://www.postgresql.org/docs/9.6/pgbuffercache.html)               | 1.2             | Examina la caché del búfer compartido|
> |[pg_partman](https://github.com/pgpartman/pg_partman)                   | 2.6.3           | Extensión para administrar tablas con particiones por hora o identificador|
> |[pg_prewarm](https://www.postgresql.org/docs/9.6/pgprewarm.html)                   | 1.1             | Prepara los datos de relación|
> |[pg_stat_statements](https://www.postgresql.org/docs/9.6/pgstatstatements.html)           | 1.4             | Realiza un seguimiento de las estadísticas de ejecución de todas las instrucciones SQL ejecutadas|
> |[pg_trgm](https://www.postgresql.org/docs/9.6/pgtrgm.html)                      | 1.3             | Medición de similitud de texto y búsqueda de índice basada en trigramas|
> |[plpgsql](https://www.postgresql.org/docs/9.6/plpgsql.html)                      | 1.0             | Lenguaje de procedimientos de PL/pgSQL|
> |[plv8](https://plv8.github.io/)                         | 2.1.0          | Lenguaje de procedimientos de confianza de PL/JavaScript (v8)|
> |[postgis](https://www.postgis.net/)                      | 2.3.2           | Geometría, geografía y funciones y tipos espaciales de trama de PostGIS|
> |[postgis_sfcgal](https://www.postgis.net/)               | 2.3.2           | Funciones de PostGIS SFCGAL|
> |[postgis_tiger_geocoder](https://www.postgis.net/)       | 2.3.2           | Geocoder de PostGIS tiger y geocoder inverso|
> |[postgis_topology](https://postgis.net/docs/Topology.html)             | 2.3.2           | Funciones y tipos espaciales de topología PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/9.6/postgres-fdw.html)                 | 1.0             | Contenedor de datos externos para servidores PostgreSQL remotos|
> |[tablefunc](https://www.postgresql.org/docs/9.6/tablefunc.html)                    | 1.0             | Funciones que manipulan la totalidad del contenido de las tablas, incluidas tablas de referencias cruzadas|
> |[timescaledb](https://docs.timescale.com/latest)                    | 1.7.4             | Habilita las inserciones escalables y las consultas complejas para los datos de serie temporal|
> |[unaccent](https://www.postgresql.org/docs/9.6/unaccent.html)                     | 1.1             | Diccionario de búsqueda de texto que quita los acentos|
> |[uuid-ossp](https://www.postgresql.org/docs/9.6/uuid-ossp.html)                    | 1.1             | Genera identificadores únicos universales (UUID)|

## <a name="postgres-95-extensions"></a>Extensiones de Postgres 9.5

>[!NOTE]
> Se ha retirado PostgreSQL, versión 9.5.

Las extensiones siguientes están disponibles en los servidores de Azure Database for PostgreSQL que tienen la versión 9.5 de Postgres.

> [!div class="mx-tableFixed"]
> | **Extensión**| **Versión de la extensión** | **Descripción** |
> |---|---|---|
> |[address_standardizer](http://postgis.net/docs/Address_Standardizer.html)         | 2.3.0           | Se utilizan para analizar una dirección en los elementos que la componen. |
> |[address_standardizer_data_us](http://postgis.net/docs/Address_Standardizer.html) | 2.3.0           | Aborda el ejemplo del conjunto de datos estandarizado de EE. UU.|
> |[btree_gin](https://www.postgresql.org/docs/9.5/btree-gin.html)                    | 1.0             | Compatibilidad con la indexación de tipos de datos comunes en GIN|
> |[btree_gist](https://www.postgresql.org/docs/9.5/btree-gist.html)                   | 1.1             | Compatibilidad con la indexación de tipos de datos comunes en GiST|
> |[chkpass](https://www.postgresql.org/docs/9.5/chkpass.html)                       | 1.0             | Tipo de datos para las contraseñas de cifrado automático|
> |[citext](https://www.postgresql.org/docs/9.5/citext.html)                       | 1.1             | Tipo de datos para cadenas de caracteres que no distinguen mayúsculas de minúsculas|
> |[cube](https://www.postgresql.org/docs/9.5/cube.html)                         | 1.0             | Tipo de datos para los cubos multidimensionales|
> |[dblink](https://www.postgresql.org/docs/9.5/dblink.html)                       | 1.1             | Se conecta a otras bases de datos de PostgreSQL desde una base de datos|
> |[dict_int](https://www.postgresql.org/docs/9.5/dict-int.html)                     | 1.0             | Plantilla de diccionario de búsqueda de texto para números enteros|
> |[earthdistance](https://www.postgresql.org/docs/9.5/earthdistance.html)                | 1.0             | Calcula distancias de círculo máximo en la superficie de la Tierra|
> |[fuzzystrmatch](https://www.postgresql.org/docs/9.5/fuzzystrmatch.html)                | 1.0             | Determina las similitudes y la distancia entre las cadenas|
> |[hstore](https://www.postgresql.org/docs/9.5/hstore.html)                       | 1.3             | Tipo de datos para almacenar conjuntos de pares (clave/valor)|
> |[hypopg](https://hypopg.readthedocs.io/en/latest/)                       | 1.1.1           | Índices hipotéticos para PostgreSQL|
> |[intarray](https://www.postgresql.org/docs/9.5/intarray.html)                     | 1.0             | Funciones, operadores e índices compatibles con matrices 1D de números enteros|
> |[isn](https://www.postgresql.org/docs/9.5/isn.html)                          | 1.0             | Tipos de datos para los estándares internacionales de numeración de productos|
> |[ltree](https://www.postgresql.org/docs/9.5/ltree.html)                        | 1.0             | Tipo de datos para las estructuras de árbol jerárquicas|
> |[orafce](https://github.com/orafce/orafce)                       | 3.7             | Funciones y operadores que emulan un subconjunto de funciones y paquetes desde un RDBMS comercial|
> |[pgaudit](https://www.pgaudit.org/)                     | 1.0.7             | Proporciona funcionalidad de auditoría|
> |[pgcrypto](https://www.postgresql.org/docs/9.5/pgcrypto.html)                     | 1.2             | Funciones de cifrado|
> |[pgrouting](https://pgrouting.org/)                    | 2.3.0           | Extensión pgRouting|
> |[pgrowlocks](https://www.postgresql.org/docs/9.5/pgrowlocks.html)                   | 1.1             | Muestra información de bloqueo de nivel de fila|
> |[pgstattuple](https://www.postgresql.org/docs/9.5/pgstattuple.html)                  | 1.3             | Muestra estadísticas de nivel de tupla|
> |[pg_buffercache](https://www.postgresql.org/docs/9.5/pgbuffercache.html)               | 1.1             | Examina la caché del búfer compartido|
> |[pg_partman](https://github.com/pgpartman/pg_partman)                   | 2.6.3           | Extensión para administrar tablas con particiones por hora o identificador|
> |[pg_prewarm](https://www.postgresql.org/docs/9.5/pgprewarm.html)                   | 1.0             | Prepara los datos de relación|
> |[pg_stat_statements](https://www.postgresql.org/docs/9.5/pgstatstatements.html)           | 1.3             | Realiza un seguimiento de las estadísticas de ejecución de todas las instrucciones SQL ejecutadas|
> |[pg_trgm](https://www.postgresql.org/docs/9.5/pgtrgm.html)                      | 1.1             | Medición de similitud de texto y búsqueda de índice basada en trigramas|
> |[plpgsql](https://www.postgresql.org/docs/9.5/plpgsql.html)                      | 1.0             | Lenguaje de procedimientos de PL/pgSQL|
> |[postgis](https://www.postgis.net/)                      | 2.3.0           | Geometría, geografía y funciones y tipos espaciales de trama de PostGIS|
> |[postgis_sfcgal](https://www.postgis.net/)               | 2.3.0           | Funciones de PostGIS SFCGAL|
> |[postgis_tiger_geocoder](https://www.postgis.net/)       | 2.3.0           | Geocoder de PostGIS tiger y geocoder inverso|
> |[postgis_topology](https://postgis.net/docs/Topology.html)             | 2.3.0           | Funciones y tipos espaciales de topología PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/9.5/postgres-fdw.html)                 | 1.0             | Contenedor de datos externos para servidores PostgreSQL remotos|
> |[tablefunc](https://www.postgresql.org/docs/9.5/tablefunc.html)                    | 1.0             | Funciones que manipulan la totalidad del contenido de las tablas, incluidas tablas de referencias cruzadas|
> |[unaccent](https://www.postgresql.org/docs/9.5/unaccent.html)                     | 1.0             | Diccionario de búsqueda de texto que quita los acentos|
> |[uuid-ossp](https://www.postgresql.org/docs/9.5/uuid-ossp.html)                    | 1.0             | Genera identificadores únicos universales (UUID)|


## <a name="pg_stat_statements"></a>pg_stat_statements
La [extensión pg_stat_statements](https://www.postgresql.org/docs/current/pgstatstatements.html) está cargada previamente en cada servidor de Azure Database for PostgreSQL para proporcionarle un medio de seguimiento de las estadísticas de ejecución de las instrucciones SQL.
La configuración `pg_stat_statements.track`, que controla las instrucciones que la extensión cuenta, se establece de manera predeterminada en `top`, lo que significa que se realiza el seguimiento de todas las instrucciones que los clientes emiten directamente. Los otros dos niveles de seguimiento son `none` y `all`. Esta configuración es configurable como un parámetro de servidor a través de [Azure Portal](./howto-configure-server-parameters-using-portal.md) o la [CLI de Azure](./howto-configure-server-parameters-using-cli.md).

Hay un equilibrio entre la información de ejecución de consulta que pg_stat_statements proporciona y el impacto en el rendimiento del servidor al registrar cada instrucción SQL. Si no está usando activamente la extensión pg_stat_statements, le recomendamos que establezca `pg_stat_statements.track` en `none`. Tenga en cuenta que algunos servicios de supervisión de terceros pueden basarse en pg_stat_statements para entregar información de rendimiento de consultas, por lo que debe confirmar si este es el caso para usted o no.

## <a name="dblink-and-postgres_fdw"></a>dblink y postgres_fdw
[dblink](https://www.postgresql.org/docs/current/contrib-dblink-function.html) y [postgres_fdw](https://www.postgresql.org/docs/current/postgres-fdw.html) le permiten conectarse de un servidor PostgreSQL a otro, o a otra base de datos en el mismo servidor. El servidor de recepción debe permitir conexiones del servidor de envío a través de su firewall. Cuando se usan estas extensiones para conectarse entre servidores de Azure Database for PostgreSQL, esto se puede realizar mediante el establecimiento de "Permitir acceso a servicios de Azure" en Activado. Esto también es necesario si desea utilizar las extensiones para volver al mismo servidor. La opción "Permitir el acceso a servicios de Azure" puede encontrarse en la página de Azure Portal para el servidor de Postgres, en Seguridad de la conexión. Al activar "Permitir el acceso a servicios de Azure", se colocan todas las direcciones IP de Azure en la lista de permitidos.

En la actualidad, no se admiten las conexiones salientes desde Azure Database for PostgreSQL, excepto para las conexiones a otros servidores de Azure Database for PostgreSQL en la misma región.

## <a name="uuid"></a>uuid
Si planea usar `uuid_generate_v4()` desde la [extensión uuid-ossp](https://www.postgresql.org/docs/current/uuid-ossp.html), puede comparar con `gen_random_uuid()` de la [extensión pgcrypto](https://www.postgresql.org/docs/current/pgcrypto.html) para obtener beneficios de rendimiento.

## <a name="pgaudit"></a>pgAudit
La [extensión pgAudit](https://github.com/pgaudit/pgaudit/blob/master/README.md) proporciona registro de auditoría de objetos y de sesiones. Para aprender a usar esta extensión en Azure Database for PostgreSQL, visite el [artículo acerca de los conceptos de auditoría](concepts-audit.md). 

## <a name="pg_prewarm"></a>pg_prewarm
La extensión pg_prewarm carga los datos relacionales en la memoria caché. El precalentamiento de las memorias caché significa que las consultas tengan mejores tiempos de respuesta en su primera ejecución después de un reinicio. En Postgres 10 y versiones anteriores, el precalentamiento se realiza manualmente mediante la [función prewarm](https://www.postgresql.org/docs/10/pgprewarm.html).

En Postgres 11 y versiones posteriores, puede configurar el precalentamiento para que se produzca [automáticamente](https://www.postgresql.org/docs/current/pgprewarm.html). Debe incluir pg_prewarm en la lista de parámetros de `shared_preload_libraries` y reiniciar el servidor para aplicar el cambio. Los parámetros se pueden establecer desde [Azure Portal](howto-configure-server-parameters-using-portal.md), la [CLI](howto-configure-server-parameters-using-cli.md), la API de REST o una plantilla de ARM. 

## <a name="timescaledb"></a>TimescaleDB
TimescaleDB es una base de datos de serie temporal que se empaqueta como una extensión para PostgreSQL. TimescaleDB proporciona funciones analíticas orientadas al tiempo, optimizaciones y escala Postgres para las cargas de trabajo de serie temporal.

[Más información sobre TimescaleDB](https://docs.timescale.com/latest), una marca registrada de [Timescale, Inc.](https://www.timescale.com/) Azure Database for PostgreSQL proporciona la [edición Apache-2](https://www.timescale.com/legal/licenses) de TimescaleDB.

### <a name="installing-timescaledb"></a>Instalación de TimescaleDB
Para instalar TimescaleDB, tendrá que incluirlo en las bibliotecas de carga previa compartidas del servidor. Si cambia el parámetro `shared_preload_libraries` de Postgres deberá **reiniciar el servidor** para que tenga efecto. Puede cambiar los parámetros mediante [Azure Portal](howto-configure-server-parameters-using-portal.md) o la [CLI de Azure](howto-configure-server-parameters-using-cli.md).

Mediante [Azure Portal](https://portal.azure.com/):

1. Seleccione su servidor de Azure Database for PostgreSQL.

2. En la barra lateral, seleccione **Parámetros del servidor**.

3. Busque el parámetro `shared_preload_libraries`.

4. Seleccione **TimescaleDB**.

5. Seleccione **Guardar** para conservar los cambios. Recibirá una notificación una vez que se haya guardado el cambio. 

6. Después de la notificación, **reinicie** el servidor para aplicar estos cambios. Para aprender a reiniciar un servidor, consulte [Reinicio de un servidor de Azure Database for PostgreSQL mediante Azure Portal](howto-restart-server-portal.md).


Ahora puede habilitar TimescaleDB en la base de datos de Postgres. Conéctese a la base de datos y ejecute el comando siguiente:
```sql
CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;
```
> [!TIP]
> Si ve un error, confirme que ha [reiniciado el servidor](howto-restart-server-portal.md) después de guardar shared_preload_libraries. 

Ahora puede crear una hipertabla de TimescaleDB [desde cero](https://docs.timescale.com/getting-started/creating-hypertables) o migrar [datos de serie temporal existentes en PostgreSQL](https://docs.timescale.com/getting-started/migrating-data).

### <a name="restoring-a-timescale-database"></a>Restauración de una base de datos de escala de Timescale
Para restaurar una base de datos de Timescale mediante pg_dump y pg_restore, tiene que ejecutar dos procedimientos auxiliares en la base de datos de destino: `timescaledb_pre_restore()` y `timescaledb_post restore()`.

Primero prepare la base de datos de destino:

```SQL
--create the new database where you'll perform the restore
CREATE DATABASE tutorial;
\c tutorial --connect to the database 
CREATE EXTENSION timescaledb;

SELECT timescaledb_pre_restore();
```

Ahora puede ejecutar pg_dump en la base de datos original y, a continuación, pg_restore. Después de la restauración, asegúrese de ejecutar el siguiente comando en la base de datos restaurada:

```SQL
SELECT timescaledb_post_restore();
```


## <a name="next-steps"></a>Pasos siguientes
Si no ve una extensión que le gustaría utilizar, háganoslo saber. Vote por las solicitudes existentes o cree nuevos comentarios y solicitudes en nuestro [foro de comentarios](https://feedback.azure.com/forums/597976-azure-database-for-postgresql).