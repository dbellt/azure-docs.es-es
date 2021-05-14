---
title: 'Almacén de consultas en Azure Database for PostgreSQL: servidor flexible'
description: En este artículo se describe la característica Almacén de consultas en Azure Database for PostgreSQL con un servidor flexible.
author: ssen-msft
ms.author: ssen
ms.service: postgresql
ms.topic: conceptual
ms.date: 04/01/2021
ms.openlocfilehash: 6ab2ea6f6544bcf561e92f00b5afee693393c157
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107558783"
---
# <a name="monitor-performance-with-query-store"></a>Supervisión del rendimiento con el Almacén de consultas
**Se aplica a:** Azure Database for PostgreSQL: versiones de servidor flexible 11 y posteriores

La característica Almacén de consultas de Azure Database for PostgreSQL proporciona una manera de realizar un seguimiento del rendimiento de las consultas a lo largo del tiempo. El Almacén de consultas simplifica la solución de problemas de rendimiento al ayudar a encontrar rápidamente las consultas que tardan más en ejecutarse y consumen más recursos. El Almacén de consultas captura automáticamente un historial de consultas y estadísticas de tiempo de ejecución y lo conserva para su revisión. Segmenta los datos por tiempo para que pueda ver los patrones de uso temporales. Los datos de todos los usuarios, las bases de datos y las consultas se almacenan en una base de datos denominada **azure_sys** en la instancia de Azure Database for PostgreSQL.

> [!IMPORTANT]
> No modifique la base de datos **azure_sys** ni su esquema. Si lo hace, impedirá que el Almacén de consultas y las características de rendimiento relacionados funcionen correctamente.
## <a name="enabling-query-store"></a>Habilitación del Almacén de consultas
El Almacén de consultas es una característica opcional, por lo que no está habilitado de forma predeterminada en un servidor. El Almacén de consultas se habilita o deshabilita globalmente para todas las bases de datos en un servidor determinado y no se puede activar o desactivar por base de datos.
### <a name="enable-query-store-using-the-azure-portal"></a>Habilitación del Almacén de consultas mediante Azure Portal
1. Inicie sesión en Azure Portal y seleccione el servidor de Azure Database for PostgreSQL.
2. Seleccione **Parámetros del servidor** en la sección **Configuración** del menú.
3. Busque el parámetro `pg_qs.query_capture_mode`.
4. Establezca el valor en `TOP` o `ALL` y haga clic en **Guardar**.
Espere hasta 20 minutos para que el primer lote de datos se conserve en la base de datos azure_sys.
## <a name="information-in-query-store"></a>Información del Almacén de consultas
El Almacén de consultas consta de un único almacén:
- Un almacén de estadísticas de ejecución para conservar la información de estadísticas de ejecución de consultas.

Algunos escenarios habituales para usar el Almacén de consultas son:
- Determinar el número de veces que se ha ejecutado una consulta en un período determinado
- Comparar el tiempo de ejecución medio de una consulta a través de ventanas de tiempo para ver diferenciales de gran tamaño
- Identificar las consultas con una ejecución más larga en las últimas horas. Para minimizar el uso de espacio, se agregan las estadísticas de ejecución en tiempo de ejecución en el almacén de estadísticas de ejecución en un período fijo y configurable. La información de estos almacenes se puede consultar mediante vistas.
## <a name="access-query-store-information"></a>Acceso a información del Almacén de consultas
Los datos del Almacén de consultas se almacenan en la base de datos azure_sys del servidor de Postgres. La consulta siguiente devuelve información sobre las consultas en el Almacén de consultas:
```sql
SELECT * FROM query_store.qs_view; 
``` 

## <a name="configuration-options"></a>Opciones de configuración
Cuando el Almacén de consultas está habilitado, guarda los datos en ventanas de agregación de 15 minutos, hasta 500 consultas diferentes en cada ventana. Las siguientes opciones están disponibles para configurar los parámetros del Almacén de consultas.

| **Parámetro** | **Descripción** | **Valor predeterminado** | **Range**|
|---|---|---|---|
| pg_qs.query_capture_mode | Establece las instrucciones de las que se realiza el seguimiento. | None | none, top, all |
| pg_qs.max_query_text_length | Establece la longitud máxima de consulta que se puede guardar. Las consultas más largas se truncarán. | 6000 | 100 - 10K |
| pg_qs.retention_period_in_days | Establece el período de retención. | 7 | 1 - 30 |
| pg_qs.track_utility | Establece si se realiza un seguimiento de los comandos de la utilidad. | en | on, off |

Use [Azure Portal](howto-configure-server-parameters-using-portal.md) para obtener o establecer otro valor para un parámetro.

## <a name="views-and-functions"></a>Funciones y vistas
Vea y administre el Almacén de consultas mediante las siguientes vistas y funciones. Cualquier usuario en el rol público PostgreSQL puede utilizar estas vistas para ver los datos en el Almacén de consultas. Estas vistas solo están disponibles en la base de datos **azure_sys**.
Las consultas se normalizan examinando su estructura después de quitar los literales y constantes. Si dos consultas son idénticas, salvo por los valores literales, tendrán el mismo queryId.
### <a name="query_storeqs_view"></a>query_store.qs_view
Esta vista devuelve todos los datos del Almacén de consultas. Hay una fila por cada identificador de base de datos, de usuario y de consulta diferentes. 

|**Nombre**   |**Tipo** | **Referencias**  | **Descripción**|
|---|---|---|---|
|runtime_stats_entry_id |bigint | | Id. de la tabla runtime_stats_entries|
|user_id    |oid    |pg_authid.oid  |OID del usuario que ha ejecutado la instrucción.|
|db_id  |oid    |pg_database.oid    |OID de la base de datos en la que se ha ejecutado la instrucción.|
|query_id   |bigint  || Código hash interno, calculado a partir del árbol de análisis de la instrucción.|
|query_sql_text |Varchar(10000)  || Texto de una instrucción representativa. Las consultas diferentes con la misma estructura se agrupan; este texto es el texto para la primera consulta del clúster.|
|plan_id    |bigint |   |Identificador del plan correspondiente a esta consulta, no está disponible todavía.|
|start_time |timestamp  ||  Las consultas se agregan por ciclos: el intervalo de tiempo de un ciclo es de 15 minutos de forma predeterminada. Se trata de la hora de inicio correspondiente al ciclo para esta entrada.|
|end_time   |timestamp  ||  Hora de finalización correspondiente al ciclo para esta entrada.|
|calls  |bigint  || Número de veces que se ejecuta la consulta.|
|total_time |double precision   ||  Tiempo total de ejecución de las consultas, en milisegundos.|
|min_time   |double precision   ||  Tiempo mínimo de ejecución de las consultas, en milisegundos.|
|max_time   |double precision   ||  Tiempo máximo de ejecución de las consultas, en milisegundos.|
|mean_time  |double precision   ||  Tiempo medio de ejecución de las consultas, en milisegundos.|
|stddev_time|   double precision    ||  Desviación estándar del tiempo de ejecución de las consultas, en milisegundos. |
|rows   |bigint ||  Número total de filas recuperadas o afectadas por la instrucción.|
|shared_blks_hit|   bigint  ||  Número total de aciertos de caché de bloque compartidos por la instrucción.|
|shared_blks_read|  bigint  ||  Número total de bloques compartidos leídos por la instrucción.|
|shared_blks_dirtied|   bigint   || Número total de bloques compartidos desfasados por la instrucción. |
|shared_blks_written|   bigint  ||  Número total de bloques compartidos escritos por la instrucción.|
|local_blks_hit|    bigint ||   Número total de aciertos de caché de bloque locales por la instrucción.|
|local_blks_read|   bigint   || Número total de bloques locales leídos por la instrucción.|
|local_blks_dirtied|    bigint  ||  Número total de bloques locales desfasados por la instrucción.|
|local_blks_written|    bigint  ||  Número total de bloques locales escritos por la instrucción.|
|temp_blks_read |bigint  || Número total de bloques temporales leídos por la instrucción.|
|temp_blks_written| bigint   || Número total de bloques temporales escritos por la instrucción.|
|blk_read_time  |double precision    || Tiempo total que la instrucción dedica a leer los bloques, en milisegundos (si está habilitado track_io_timing; de lo contrario, cero).|
|blk_write_time |double precision    || Tiempo total que la instrucción dedica a escribir los bloques, en milisegundos (si está habilitado track_io_timing; de lo contrario, cero).|
    
### <a name="query_storequery_texts_view"></a>query_store.query_texts_view
Esta vista devuelve datos de texto de consulta en el Almacén de consultas. Hay una fila para cada argumento de consulta diferente.

| **Nombre** | **Tipo** | **Descripción** |
|--|--|--|
| query_text_id | bigint | Identificador de la tabla query_texts. |
| query_sql_text | Varchar(10000) | Texto de una instrucción representativa. Las consultas diferentes con la misma estructura se agrupan; este texto es el texto para la primera consulta del clúster. |

### <a name="functions"></a>Functions
`qs_reset` descarta todas las estadísticas recopiladas hasta ahora por el Almacén de consultas. Esta función solo se puede ejecutar mediante el rol de administrador del servidor.

`staging_data_reset` descarta todas las estadística recopiladas en la memoria por el Almacén de consultas (es decir, los datos en la memoria que no se ha vaciado aún a la base de datos). Esta función solo se puede ejecutar mediante el rol de administrador del servidor.

## <a name="limitations-and-known-issues"></a>Limitaciones y problemas conocidos
- Si un servidor de PostgreSQL tiene el parámetro default_transaction_read_only activado, el Almacén de consultas no podrá capturar datos.
## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre los [escenarios en los que el Almacén de consultas puede ser especialmente útil](concepts-query-store-scenarios.md).
- Más información sobre los [procedimientos recomendados para el uso del Almacén de consultas](concepts-query-store-best-practices.md)
