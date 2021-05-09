---
title: 'Extensiones en Azure Database for PostgreSQL: Servidor flexible'
description: 'Obtenga información sobre las extensiones de Postgres disponibles en Azure Database for PostgreSQL: Servidor flexible'
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.topic: conceptual
ms.date: 04/22/2021
ms.openlocfilehash: 9fdc2d2567e6ea24a68398139007be978cb639b5
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107989367"
---
# <a name="postgresql-extensions-in-azure-database-for-postgresql---flexible-server"></a>Extensiones de PostgreSQL en Azure Database for PostgreSQL: Servidor flexible

> [!IMPORTANT]
> Azure Database for PostgreSQL: Servidor flexible está en versión preliminar.

PostgreSQL ofrece la capacidad de ampliar la funcionalidad de su base de datos mediante extensiones. Las extensiones agrupan varios objetos SQL relacionados en un solo paquete que se puede cargar o quitar de la base de datos con un comando. Después de cargarse en la base de datos, las extensiones funcionan como características integradas.

## <a name="how-to-use-postgresql-extensions"></a>¿Cómo se utilizan las extensiones de PostgreSQL?
Para poder usar las extensiones de PostgreSQL, es preciso que estén instaladas en su base de datos. Para instalar una extensión determinada, ejecute el comando [CREATE EXTENSION](https://www.postgresql.org/docs/current/sql-createextension.html). Este comando carga los objetos empaquetados en la base de datos.

Azure Database for PostgreSQL admite un subconjunto de extensiones de claves, como se enumeran a continuación. Esta información también está disponible al ejecutar `SHOW azure.extensions;`. Las extensiones no enumeradas en este documento no se admiten en Azure Database for PostgreSQL: Servidor flexible. No puede crear o cargar su propia extensión en el servicio Azure Database for PostgreSQL.


## <a name="postgres-12-extensions"></a>Extensiones de Postgres 12

Las extensiones siguientes están disponibles en los servidores flexibles de Azure Database for PostgreSQL que tienen la versión 12 de Postgres. 

> [!div class="mx-tableFixed"]
> | **Extensión**| **Versión de la extensión** | **Descripción** |
> |---|---|---|
> |[address_standardizer](http://postgis.net/docs/Address_Standardizer.html)         | 3.0.0           | Se utilizan para analizar una dirección en los elementos que la componen. |
> |[address_standardizer_data_us](http://postgis.net/docs/Address_Standardizer.html) | 3.0.0           | Aborda el ejemplo del conjunto de datos estandarizado de EE. UU.|
> |[amcheck](https://www.postgresql.org/docs/12/amcheck.html)                    | 1.2             | Funciones de comprobación de la integridad de la relación.|
> |[bloom](https://www.postgresql.org/docs/12/bloom.html)                    | 1.0             | Método de acceso de bloom: índice basado en archivos de firma.|
> |[btree_gin](https://www.postgresql.org/docs/12/btree-gin.html)                    | 1.3             | Compatibilidad con la indexación de tipos de datos comunes en GIN|
> |[btree_gist](https://www.postgresql.org/docs/12/btree-gist.html)                   | 1.5             | Compatibilidad con la indexación de tipos de datos comunes en GiST|
> |[citext](https://www.postgresql.org/docs/12/citext.html)                       | 1.6             | Tipo de datos para cadenas de caracteres que no distinguen mayúsculas de minúsculas|
> |[cube](https://www.postgresql.org/docs/12/cube.html)                         | 1.4             | Tipo de datos para los cubos multidimensionales|
> |[dblink](https://www.postgresql.org/docs/12/dblink.html)                       | 1.2             | Se conecta a otras bases de datos de PostgreSQL desde una base de datos|
> |[dict_int](https://www.postgresql.org/docs/12/dict-int.html)                     | 1.0             | Plantilla de diccionario de búsqueda de texto para números enteros|
> |[dict_xsyn](https://www.postgresql.org/docs/12/dict-xsyn.html)                     | 1,0             | Plantilla de diccionario de búsqueda de texto para el procesamiento de sinónimos extendido.|
> |[earthdistance](https://www.postgresql.org/docs/12/earthdistance.html)                | 1.1             | Calcula distancias de círculo máximo en la superficie de la Tierra|
> |[fuzzystrmatch](https://www.postgresql.org/docs/12/fuzzystrmatch.html)                | 1.1             | Determina las similitudes y la distancia entre las cadenas|
> |[hstore](https://www.postgresql.org/docs/12/hstore.html)                       | 1.6             | Tipo de datos para almacenar conjuntos de pares (clave/valor)|
> |[intagg](https://www.postgresql.org/docs/12/intagg.html)                     | 1.1             | Agregador y enumerador de enteros. (Obsoleto)|
> |[intarray](https://www.postgresql.org/docs/12/intarray.html)                     | 1.2             | Funciones, operadores e índices compatibles con matrices 1D de números enteros|
> |[isn](https://www.postgresql.org/docs/12/isn.html)                          | 1.2             | Tipos de datos para los estándares internacionales de numeración de productos|
> |[ltree](https://www.postgresql.org/docs/12/ltree.html)                        | 1.1             | Tipo de datos para las estructuras de árbol jerárquicas|
> |[pageinspect](https://www.postgresql.org/docs/12/pageinspect.html)                        | 1.7             | Inspección del contenido de páginas de bases de datos a un nivel bajo.|
> |[pg_buffercache](https://www.postgresql.org/docs/12/pgbuffercache.html)               | 1.3             | Examina la caché del búfer compartido|
> |[pg_freespacemap](https://www.postgresql.org/docs/12/pgfreespacemap.html)               | 1.2             | Examen de la asignación de espacio libre (FSM).|
> |[pg_prewarm](https://www.postgresql.org/docs/12/pgprewarm.html)                   | 1.2             | Prepara los datos de relación|
> |[pg_stat_statements](https://www.postgresql.org/docs/12/pgstatstatements.html)           | 1.7             | Realiza un seguimiento de las estadísticas de ejecución de todas las instrucciones SQL ejecutadas|
> |[pg_trgm](https://www.postgresql.org/docs/12/pgtrgm.html)                      | 1.4             | Medición de similitud de texto y búsqueda de índice basada en trigramas|
> |[pg_visibility](https://www.postgresql.org/docs/12/pgvisibility.html)                      | 1.2             | Examen del mapa de visibilidad (VM) y la información de visibilidad de nivel de página.|
> |[pgaudit](https://www.pgaudit.org/)                     | 1.4             | Proporciona funcionalidad de auditoría|
> |[pgcrypto](https://www.postgresql.org/docs/12/pgcrypto.html)                     | 1.3             | Funciones de cifrado|
> |[pglogical](https://github.com/2ndQuadrant/pglogical)                     | 2.3.2           | Replicación lógica de PostgreSQL|
> |[pgrowlocks](https://www.postgresql.org/docs/12/pgrowlocks.html)                   | 1.2             | Muestra información de bloqueo de nivel de fila|
> |[pgstattuple](https://www.postgresql.org/docs/12/pgstattuple.html)                  | 1.5             | Muestra estadísticas de nivel de tupla|
> |[plpgsql](https://www.postgresql.org/docs/12/plpgsql.html)                      | 1.0             | Lenguaje de procedimientos de PL/pgSQL|
> |[postgis](https://www.postgis.net/)                      | 3.0.0           | Geografía/geometría de PostGIS |
> |[postgis_raster](https://www.postgis.net/)               | 3.0.0           | Funciones y tipos de trama de PostGIS| 
> |[postgis_sfcgal](https://www.postgis.net/)               | 3.0.0           | Funciones de PostGIS SFCGAL|
> |[postgis_tiger_geocoder](https://www.postgis.net/)       | 3.0.0           | Geocoder de PostGIS tiger y geocoder inverso|
> |[postgis_topology](https://postgis.net/docs/Topology.html)             | 3.0.0           | Funciones y tipos espaciales de topología PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/12/postgres-fdw.html)                 | 1.0             | Contenedor de datos externos para servidores PostgreSQL remotos|
> |[sslinfo](https://www.postgresql.org/docs/12/sslinfo.html)                    | 1.2             | Información sobre los certificados SSL.|
> |[tsm_system_rows](https://www.postgresql.org/docs/12/tsm-system-rows.html)                    | 1.0             |  Método TABLESAMPLE que acepta el número de filas como un límite.|
> |[tsm_system_time](https://www.postgresql.org/docs/12/tsm-system-time.html)                    | 1.0             |  Método TABLESAMPLE que acepta el tiempo en milisegundos como un límite.|
> |[unaccent](https://www.postgresql.org/docs/12/unaccent.html)                     | 1.1             | Diccionario de búsqueda de texto que quita los acentos|
> |[uuid-ossp](https://www.postgresql.org/docs/12/uuid-ossp.html)                    | 1.1             | Genera identificadores únicos universales (UUID)|

## <a name="postgres-11-extensions"></a>Extensiones de Postgres 11

Las extensiones siguientes están disponibles en los servidores flexibles de Azure Database for PostgreSQL que tienen la versión 11 de Postgres. 

> [!div class="mx-tableFixed"]
> | **Extensión**| **Versión de la extensión** | **Descripción** |
> |---|---|---|
> |[address_standardizer](http://postgis.net/docs/Address_Standardizer.html)         | 2.5.1           | Se utilizan para analizar una dirección en los elementos que la componen. |
> |[address_standardizer_data_us](http://postgis.net/docs/Address_Standardizer.html) | 2.5.1           | Aborda el ejemplo del conjunto de datos estandarizado de EE. UU.|
> |[amcheck](https://www.postgresql.org/docs/11/amcheck.html)                    | 1.1             | Funciones de comprobación de la integridad de la relación.|
> |[bloom](https://www.postgresql.org/docs/11/bloom.html)                    | 1.0             | Método de acceso de bloom: índice basado en archivos de firma.|
> |[btree_gin](https://www.postgresql.org/docs/11/btree-gin.html)                    | 1.3             | Compatibilidad con la indexación de tipos de datos comunes en GIN|
> |[btree_gist](https://www.postgresql.org/docs/11/btree-gist.html)                   | 1.5             | Compatibilidad con la indexación de tipos de datos comunes en GiST|
> |[citext](https://www.postgresql.org/docs/11/citext.html)                       | 1.5             | Tipo de datos para cadenas de caracteres que no distinguen mayúsculas de minúsculas|
> |[cube](https://www.postgresql.org/docs/11/cube.html)                         | 1.4             | Tipo de datos para los cubos multidimensionales|
> |[dblink](https://www.postgresql.org/docs/11/dblink.html)                       | 1.2             | Se conecta a otras bases de datos de PostgreSQL desde una base de datos|
> |[dict_int](https://www.postgresql.org/docs/11/dict-int.html)                     | 1.0             | Plantilla de diccionario de búsqueda de texto para números enteros|
> |[dict_xsyn](https://www.postgresql.org/docs/11/dict-xsyn.html)                     | 1,0             | Plantilla de diccionario de búsqueda de texto para el procesamiento de sinónimos extendido.|
> |[earthdistance](https://www.postgresql.org/docs/11/earthdistance.html)                | 1.1             | Calcula distancias de círculo máximo en la superficie de la Tierra|
> |[fuzzystrmatch](https://www.postgresql.org/docs/11/fuzzystrmatch.html)                | 1.1             | Determina las similitudes y la distancia entre las cadenas|
> |[hstore](https://www.postgresql.org/docs/11/hstore.html)                       | 1.5             | Tipo de datos para almacenar conjuntos de pares (clave/valor)|
> |[intagg](https://www.postgresql.org/docs/11/intagg.html)                     | 1.1             | Agregador y enumerador de enteros. (Obsoleto)|
> |[intarray](https://www.postgresql.org/docs/11/intarray.html)                     | 1.2             | Funciones, operadores e índices compatibles con matrices 1D de números enteros|
> |[isn](https://www.postgresql.org/docs/11/isn.html)                          | 1.2             | Tipos de datos para los estándares internacionales de numeración de productos|
> |[ltree](https://www.postgresql.org/docs/11/ltree.html)                        | 1.1             | Tipo de datos para las estructuras de árbol jerárquicas|
> |[pageinspect](https://www.postgresql.org/docs/11/pageinspect.html)                        | 1.7             | Inspección del contenido de páginas de bases de datos a un nivel bajo.|
> |[pg_buffercache](https://www.postgresql.org/docs/11/pgbuffercache.html)               | 1.3             | Examina la caché del búfer compartido|
> |[pg_freespacemap](https://www.postgresql.org/docs/11/pgfreespacemap.html)               | 1.2             | Examen de la asignación de espacio libre (FSM).|
> |[pg_prewarm](https://www.postgresql.org/docs/11/pgprewarm.html)                   | 1.2             | Prepara los datos de relación|
> |[pg_stat_statements](https://www.postgresql.org/docs/11/pgstatstatements.html)           | 1.6             | Realiza un seguimiento de las estadísticas de ejecución de todas las instrucciones SQL ejecutadas|
> |[pg_trgm](https://www.postgresql.org/docs/11/pgtrgm.html)                      | 1.4             | Medición de similitud de texto y búsqueda de índice basada en trigramas|
> |[pg_visibility](https://www.postgresql.org/docs/11/pgvisibility.html)                      | 1.2             | Examen del mapa de visibilidad (VM) y la información de visibilidad de nivel de página.|
> |[pgaudit](https://www.pgaudit.org/)                     | 1.3.1             | Proporciona funcionalidad de auditoría|
> |[pgcrypto](https://www.postgresql.org/docs/11/pgcrypto.html)                     | 1.3             | Funciones de cifrado|
> |[pglogical](https://github.com/2ndQuadrant/pglogical)                     | 2.3.2           | Replicación lógica de PostgreSQL|
> |[pgrowlocks](https://www.postgresql.org/docs/11/pgrowlocks.html)                   | 1.2             | Muestra información de bloqueo de nivel de fila|
> |[pgstattuple](https://www.postgresql.org/docs/11/pgstattuple.html)                  | 1.5             | Muestra estadísticas de nivel de tupla|
> |[plpgsql](https://www.postgresql.org/docs/11/plpgsql.html)                      | 1.0             | Lenguaje de procedimientos de PL/pgSQL|
> |[postgis](https://www.postgis.net/)                      | 2.5.1           | Geometría, geografía y funciones y tipos espaciales de trama de PostGIS|
> |[postgis_sfcgal](https://www.postgis.net/)               | 2.5.1           | Funciones de PostGIS SFCGAL|
> |[postgis_tiger_geocoder](https://www.postgis.net/)       | 2.5.1           | Geocoder de PostGIS tiger y geocoder inverso|
> |[postgis_topology](https://postgis.net/docs/Topology.html)             | 2.5.1           | Funciones y tipos espaciales de topología PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/11/postgres-fdw.html)                 | 1.0             | Contenedor de datos externos para servidores PostgreSQL remotos|
> |[sslinfo](https://www.postgresql.org/docs/11/sslinfo.html)                    | 1.2             | Información sobre los certificados SSL.|
> |[tablefunc](https://www.postgresql.org/docs/11/tablefunc.html)                    | 1.0             | Funciones que manipulan la totalidad del contenido de las tablas, incluidas tablas de referencias cruzadas|
> |[tsm_system_rows](https://www.postgresql.org/docs/11/tsm-system-rows.html)                    | 1.0             |  Método TABLESAMPLE que acepta el número de filas como un límite.|
> |[tsm_system_time](https://www.postgresql.org/docs/11/tsm-system-time.html)                    | 1.0             |  Método TABLESAMPLE que acepta el tiempo en milisegundos como un límite.|
> |[unaccent](https://www.postgresql.org/docs/11/unaccent.html)                     | 1.1             | Diccionario de búsqueda de texto que quita los acentos|
> |[uuid-ossp](https://www.postgresql.org/docs/11/uuid-ossp.html)                    | 1.1             | Genera identificadores únicos universales (UUID)|


## <a name="dblink-and-postgres_fdw"></a>dblink y postgres_fdw
[dblink](https://www.postgresql.org/docs/current/contrib-dblink-function.html) y [postgres_fdw](https://www.postgresql.org/docs/current/postgres-fdw.html) le permiten conectarse de un servidor PostgreSQL a otro, o a otra base de datos en el mismo servidor. El servidor flexible admite conexiones entrantes y salientes a cualquier servidor PostgreSQL. El servidor de envío debe permitir conexiones de salida al servidor de recepción. Del mismo modo, el servidor de recepción debe permitir conexiones del servidor de envío. 

Recomendamos implementar los servidores con la [integración con red virtual](concepts-networking.md) si tiene previsto usar estas dos extensiones. De forma predeterminada, la integración con red virtual permite conexiones entre servidores en la red virtual. También puede elegir usar [grupos de seguridad de red (red virtual)](../../virtual-network/manage-network-security-group.md) para personalizar el acceso.

## <a name="pg_prewarm"></a>pg_prewarm

La extensión pg_prewarm carga los datos relacionales en la memoria caché. El precalentamiento de las memorias caché significa que las consultas tengan mejores tiempos de respuesta en su primera ejecución después de un reinicio. La función de precalentamiento automático no está disponible actualmente en Azure Database for PostgreSQL: Servidor flexible.

## <a name="pg_stat_statements"></a>pg_stat_statements
La [extensión pg_stat_statements](https://www.postgresql.org/docs/current/pgstatstatements.html) está cargada previamente en cada servidor flexible de Azure Database for PostgreSQL para proporcionarle un medio de seguimiento de las estadísticas de ejecución de las instrucciones SQL.
La configuración `pg_stat_statements.track`, que controla las instrucciones que la extensión cuenta, se establece de manera predeterminada en `top`, lo que significa que se realiza el seguimiento de todas las instrucciones que los clientes emiten directamente. Los otros dos niveles de seguimiento son `none` y `all`. Esta configuración se puede configurar como parámetro de servidor.

Hay un equilibrio entre la información de ejecución de consulta que pg_stat_statements proporciona y el impacto en el rendimiento del servidor al registrar cada instrucción SQL. Si no está usando activamente la extensión pg_stat_statements, le recomendamos que establezca `pg_stat_statements.track` en `none`. Tenga en cuenta que algunos servicios de supervisión de terceros pueden basarse en pg_stat_statements para entregar información de rendimiento de consultas, por lo que debe confirmar si este es el caso para usted o no.


## <a name="next-steps"></a>Pasos siguientes

Si no ve una extensión que le gustaría utilizar, háganoslo saber. Vote por las solicitudes existentes o cree nuevos comentarios y solicitudes en nuestro [foro de comentarios](https://feedback.azure.com/forums/597976-azure-database-for-postgresql).