---
title: Uso de extensiones de PostgreSQL
description: Uso de extensiones de PostgreSQL
titleSuffix: Azure Arc enabled data services
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: ba92ca8a959fae389dbdb30c295e6592f76100eb
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108288533"
---
# <a name="use-postgresql-extensions-in-your-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Uso de extensiones de PostgreSQL en el grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc

PostgreSQL es idóneo cuando se usa con extensiones. De hecho, un elemento clave de la propia funcionalidad de Hiperescala es la extensión `citus` proporcionada por Microsoft, que se instala de forma predeterminada y que permite a Postgres particionar datos de manera transparente entre varios nodos.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="supported-extensions"></a>Extensiones admitidas
Las extensiones [`contrib`](https://www.postgresql.org/docs/12/contrib.html) estándar y las siguientes extensiones ya están implementadas en los contenedores del grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc:
- [`citus`](https://github.com/citusdata/citus), v: 10.0. La extensión de Citus de [Citus Data](https://www.citusdata.com/) se carga de forma predeterminada, ya que incorpora la funcionalidad de Hiperescala al motor de PostgreSQL. No se admite la eliminación de la extensión de Citus del grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc.
- [`pg_cron`](https://github.com/citusdata/pg_cron), v: 1.3
- [`pgaudit`](https://www.pgaudit.org/), v: 1.4
- plpgsql, v: 1.0
- [`postgis`](https://postgis.net), v: 3.0.2
- [`plv8`](https://plv8.github.io/), v: 2.3.14
- [`pg_partman`](https://github.com/pgpartman/pg_partman), v: 4.4.1/
- [`tdigest`](https://github.com/tvondra/tdigest), v: 1.0.1

Se publicarán actualizaciones de esta lista a medida que evolucione con el tiempo.

> [!IMPORTANT]
> Aunque puede incorporar al grupo de servidores una extensión que no sea la indicada anteriormente, en esta versión preliminar no se conservará en el sistema. Eso significa que no estará disponible después de reiniciar el sistema y tendrá que volver a incorporarla.

En esta guía se mostrará un escenario en el que se usan dos de estas extensiones:
- [`PostGIS`](https://postgis.net/)
- [`pg_cron`](https://github.com/citusdata/pg_cron)

## <a name="which-extensions-need-to-be-added-to-the-shared_preload_libraries-and-created"></a>¿Qué extensiones se deben agregar a shared_preload_libraries y cuáles se deben crear?

|Extensiones   |Es necesario agregarlas a shared_preload_libraries  |Es necesario crearlas |
|-------------|--------------------------------------------------|---------------------- |
|`pg_cron`      |No       |Sí        |
|`pg_audit`     |Sí       |Sí        |
|`plpgsql`      |Sí       |Sí        |
|`postgis`      |No       |Sí        |
|`plv8`      |No       |Sí        |

## <a name="add-extensions-to-the-shared_preload_libraries"></a>Adición de extensiones a `shared_preload_libraries`
Para obtener más información sobre `shared_preload_libraries`, lea la documentación de PostgreSQL [aquí](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-SHARED-PRELOAD-LIBRARIES):
- Este paso no es necesario para las extensiones que forman parte de `contrib`.
- Este paso no es necesario para las extensiones que shared_preload_libraries no necesita cargar previamente. En el caso de estas extensiones, puede pasar al párrafo [Creación de extensiones](#create-extensions).

### <a name="add-an-extension-at-the-creation-time-of-a-server-group"></a>Adición de una extensión en el momento de crear un grupo de servidores
```console
azdata arc postgres server create -n <name of your postgresql server group> --extensions <extension names>
```
### <a name="add-an-extension-to-an-instance-that-already-exists"></a>Adición de una extensión a una instancia que ya existe
```console
azdata arc postgres server edit -n <name of your postgresql server group> --extensions <extension names>
```




## <a name="show-the-list-of-extensions-added-to-shared_preload_libraries"></a>Mostrar la lista de extensiones agregadas a shared_preload_libraries
Luego, ejecute cualquiera de los comandos siguientes.

### <a name="with-an-azdata-cli-command"></a>Con el comando azdata de la CLI
```console
azdata arc postgres server show -n <server group name>
```
Desplácese por la salida y observe las secciones engine\extensions en las especificaciones del grupo de servidores. Por ejemplo:
```console
"engine": {
      "extensions": [
        {
          "name": "citus"
        },
        {
          "name": "pg_cron"
        }
      ]
    },
```
### <a name="with-kubectl"></a>Con kubectl
```console
kubectl describe postgresql-12s/postgres02
```
Desplácese por la salida y observe las secciones engine\extensions en las especificaciones del grupo de servidores. Por ejemplo:
```console
Engine:
    Extensions:
      Name:  citus
      Name:  pg_cron
```


## <a name="create-extensions"></a>Creación de extensiones
Conéctese al grupo de servidores con la herramienta cliente que prefiera y ejecute la consulta estándar de PostgreSQL:
```console
CREATE EXTENSION <extension name>;
```

## <a name="show-the-list-of-extensions-created"></a>Mostrar la lista de extensiones creadas
Conéctese al grupo de servidores con la herramienta cliente que prefiera y ejecute la consulta estándar de PostgreSQL:
```console
select * from pg_extension;
```

## <a name="drop-an-extension"></a>Eliminación de una extensión
Conéctese al grupo de servidores con la herramienta cliente que prefiera y ejecute la consulta estándar de PostgreSQL:
```console
drop extension <extension name>;
```

## <a name="the-postgis-extension"></a>La extensión `PostGIS`.
No es necesario agregar la extensión `PostGIS` a `shared_preload_libraries`.
Obtenga [datos de ejemplo](http://duspviz.mit.edu/tutorials/intro-postgis/) del Department of Urban Studies & Planning (Departamento de estudios y planificación urbana) del MIT. Ejecute `apt-get install unzip` para instalar unzip si es necesario.

```console
wget http://duspviz.mit.edu/_assets/data/intro-postgis-datasets.zip
unzip intro-postgis-datasets.zip
```

Ahora se conectará a la base de datos y creará la extensión `PostGIS`:

```console
CREATE EXTENSION postgis;
```

> [!NOTE]
> Si quiere usar una de las extensiones del paquete `postgis` (por ejemplo `postgis_raster`, `postgis_topology`, `postgis_sfcgal`, `fuzzystrmatch`...), primero debe crear la extensión postgis y después la otra. Por ejemplo: `CREATE EXTENSION postgis`; `CREATE EXTENSION postgis_raster`;

Y cree el esquema:

```sql
CREATE TABLE coffee_shops (
  id serial NOT NULL,
  name character varying(50),
  address character varying(50),
  city character varying(50),
  state character varying(50),
  zip character varying(10),
  lat numeric,
  lon numeric,
  geom geometry(POINT,4326)
);
CREATE INDEX coffee_shops_gist ON coffee_shops USING gist (geom);
```

Ahora, se puede combinar `PostGIS` con la funcionalidad de escalado horizontal convirtiendo la tabla coffee_shops en distribuida:

```sql
SELECT create_distributed_table('coffee_shops', 'id');
```

Se cargarán algunos datos:

```console
\copy coffee_shops(id,name,address,city,state,zip,lat,lon) from cambridge_coffee_shops.csv CSV HEADER;
```

Rellene el campo `geom` con la latitud y la longitud codificadas correctamente en el tipo de datos `geometry` de `PostGIS`:

```sql
UPDATE coffee_shops SET geom = ST_SetSRID(ST_MakePoint(lon,lat),4326);
```

Ahora se pueden enumerar las cafeterías más cercanas al MIT (77 Massachusetts Ave; 42.359055, -71.093500):

```sql
SELECT name, address FROM coffee_shops ORDER BY geom <-> ST_SetSRID(ST_MakePoint(-71.093500,42.359055),4326);
```


## <a name="the-pg_cron-extension"></a>La extensión `pg_cron`.

Ahora, vamos a habilitar `pg_cron` en el grupo de servidores de PostgreSQL, para lo cual se va a agregar a shared_preload_libraries:

```console
azdata postgres server update -n pg2 -ns arc --extensions pg_cron
```

El grupo de servidores se reiniciará para completar la instalación de las extensiones. Esta operación puede tardar entre 2 y 3 minutos.

Ahora se puede volver a conectar y crear la extensión `pg_cron`:

```sql
CREATE EXTENSION pg_cron;
```

Con fines de prueba, vamos a crear una tabla `the_best_coffee_shop` que tome un nombre aleatorio de la tabla `coffee_shops` anterior e inserte el contenido de la tabla:

```sql
CREATE TABLE the_best_coffee_shop(name text);
```

Se puede usar `cron.schedule` más algunas instrucciones SQL para obtener un nombre de tabla aleatorio (observe el uso de una tabla temporal para almacenar el resultado de una consulta distribuida) y almacenarla en `the_best_coffee_shop`:

```sql
SELECT cron.schedule('* * * * *', $$
  TRUNCATE the_best_coffee_shop;
  CREATE TEMPORARY TABLE tmp AS SELECT name FROM coffee_shops ORDER BY random() LIMIT 1;
  INSERT INTO the_best_coffee_shop SELECT * FROM tmp;
  DROP TABLE tmp;
$$);
```

Y ahora, una vez por minuto, se obtiene un nombre diferente:

```sql
SELECT * FROM the_best_coffee_shop;
```

```console
      name
-----------------
 B & B Snack Bar
(1 row)
```

Vea el archivo [LÉAME de pg_cron](https://github.com/citusdata/pg_cron) para obtener información completa sobre la sintaxis.


## <a name="next-steps"></a>Pasos siguientes
- Consulte la documentación sobre [`plv8`](https://plv8.github.io/).
- Consulte la documentación sobre [`PostGIS`](https://postgis.net/).
- Consulte la documentación sobre [`pg_cron`](https://github.com/citusdata/pg_cron).