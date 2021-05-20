---
title: 'Oracle a Azure Database for PostgreSQL: guía de migración'
titleSuffix: Azure Database for PostgreSQL
description: Esta guía sirve de ayuda para migrar el esquema de Oracle a Azure Database for PostgreSQL.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.subservice: migration-guide
ms.topic: how-to
ms.date: 03/18/2021
ms.openlocfilehash: c197f5200c3db60a63bfd8c8e4e01ab4881aa5b6
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108756366"
---
# <a name="migrate-oracle-to-azure-database-for-postgresql"></a>Migración de Oracle a Azure Database for PostgreSQL

Esta guía sirve de ayuda para migrar el esquema de Oracle a Azure Database for PostgreSQL. 

Para obtener instrucciones de migración detalladas y completas, vea los [recursos de la guía de migración](https://github.com/microsoft/OrcasNinjaTeam/blob/master/Oracle%20to%20PostgreSQL%20Migration%20Guide/Oracle%20to%20Azure%20Database%20for%20PostgreSQL%20Migration%20Guide.pdf). 

## <a name="prerequisites"></a>Requisitos previos

Para migrar el esquema de Oracle a Azure Database for PostgreSQL necesitará hacer lo siguiente: 

- Comprobar que el entorno de origen es compatible. 
- Descargar la última versión de [ora2pg](https://ora2pg.darold.net/). 
- Tener la versión más reciente del [módulo DBD](https://www.cpan.org/modules/by-module/DBD/). 


## <a name="overview"></a>Información general

PostgreSQL es una de las bases de datos de código abierto más avanzadas del mundo. En este artículo se explica cómo usar la herramienta gratuita ora2pg para migrar una base de datos de Oracle a PostgreSQL. Puede usar ora2pg para migrar una base de datos de Oracle o MySQL a un esquema compatible de PostgreSQL. 

La herramienta ora2pg conecta la base de datos de Oracle, la examina automáticamente y extrae su estructura o sus datos. Luego ora2pg genera scripts SQL que se pueden cargar en la base de datos de PostgreSQL. Puede usar ora2pg para tareas como la ingeniería inversa de una base de datos de Oracle, la migración de una base de datos empresarial de gran tamaño o simplemente la replicación de algunos datos de Oracle en una base de datos de PostgreSQL. La herramienta es fácil de usar y no requiere conocimientos sobre bases de datos de Oracle que vayan más allá de la capacidad de proporcionar los parámetros necesarios para conectarse a la base de datos de Oracle.

> [!NOTE]
> Para obtener más información acerca del uso de la versión más reciente de ora2pg, consulte la [documentación de ora2pg](https://ora2pg.darold.net/documentation.html).

### <a name="typical-ora2pg-migration-architecture"></a>Arquitectura de migración típica de ora2pg

![Captura de pantalla de la arquitectura de migración de ora2pg.](media/howto-migrate-from-oracle/ora2pg-migration-architecture.png)

Después de aprovisionar la máquina virtual y Azure Database for PostgreSQL, necesita dos configuraciones para habilitar la conectividad entre ellos: **Permitir el acceso a servicios de Azure** y **Aplicar conexión SSL**: 

- Hoja **Seguridad de conexión** -> **Permitir el acceso a servicios de Azure** > **ACTIVADO**

- Hoja **Seguridad de conexión** -> **Configuración de SSL** > **Aplicar conexión SSL** > **DESHABILITADO**

### <a name="recommendations"></a>Recomendaciones

- Para mejorar el rendimiento de las operaciones de evaluación o exportación en el servidor de Oracle, recopile estadísticas:

   ```
   BEGIN
   
     DBMS_STATS.GATHER_SCHEMA_STATS
     DBMS_STATS.GATHER_DATABASE_STATS
     DBMS_STATS.GATHER_DICTIONARY_STATS
     END;
   ```

- Exporte datos mediante el comando `COPY` en lugar de `INSERT`.

- Evite exportar tablas con sus claves externas (FK), restricciones e índices. Estos elementos ralentizan el proceso de importación de datos en PostgreSQL.

- Cree vistas materializadas mediante la *cláusula no data*. Después, actualice las vistas.

- Si es posible, use índices únicos en las vistas materializadas. Estos índices pueden acelerar la actualización cuando se usa la sintaxis `REFRESH MATERIALIZED VIEW CONCURRENTLY`.


## <a name="pre-migration"></a>Antes de la migración 

Después de comprobar que el entorno de origen es compatible y que se cumplen los requisitos previos, está listo para iniciar la fase de premigración. Para empezar: 

1. **Descubrir**: realice el inventario de las bases de datos que necesita migrar. 
2. **Evaluar:** evalúe esas bases de datos para detectar posibles problemas u obstáculos de migración.
3. **Convertir**: resuelva los problemas que haya detectado. 
 
En el caso de las migraciones heterogéneas, como Oracle a Azure Database for PostgreSQL, esta fase también implica hacer que los esquemas de las bases de datos de origen sean compatibles con el entorno de destino.

### <a name="discover"></a>Descubra

El objetivo de la fase de detección es identificar orígenes de datos existentes y detalles sobre las características que se están usando. Esta fase ayuda a entender mejor y a planear la migración. Este proceso conlleva examinar la red para identificar todas las instancias de Oracle de la organización junto con la versión y las características en uso.

Los scripts anteriores a la evaluación de Microsoft para Oracle se ejecutan en la base de datos de Oracle. Estos scripts consultan los metadatos de Oracle. Los scripts proporcionan:

- Un inventario de bases de datos, incluidos los recuentos de objetos por esquema, tipo y estado.
- Una estimación aproximada de los datos sin procesar de cada esquema basada en estadísticas.
- El tamaño de las tablas de cada esquema.
- El número de líneas de código por paquete, función, procedimiento, etc.

Descargue los scripts relacionados de [GitHub](https://github.com/microsoft/DataMigrationTeam/tree/master/Oracle%20Inventory%20Script%20Artifacts).

### <a name="assess"></a>Evaluar

Después de hacer el inventario de las bases de datos de Oracle, va a tener una idea del tamaño de las bases de datos y de los posibles desafíos. El siguiente paso es ejecutar la evaluación.

No es fácil estimar el costo de una migración de Oracle a PostgreSQL. Para evaluar el costo de la migración, ora2pg comprueba todos los objetos de base de datos, las funciones y los procedimientos almacenados de los objetos y el código PL/SQL que no puede convertir automáticamente.

La herramienta ora2pg tiene un modo de análisis de contenido que inspecciona la base de datos de Oracle para generar un informe de texto. El informe describe lo que contiene la base de datos de Oracle y lo que no se puede exportar.

Para activar el modo de *análisis e informe*, use el tipo exportado `SHOW_REPORT` como se muestra en el siguiente comando:

```
ora2pg -t SHOW_REPORT
```

La herramienta ora2pg puede convertir código SQL y PL/SQL de la sintaxis de Oracle a PostgreSQL. Así, una vez analizada la base de datos, ora2pg puede estimar las dificultades del código y el tiempo necesario para migrar una base de datos completa.

Para estimar el costo de la migración en días por persona, ora2pg permite usar una directiva de configuración de nombre `ESTIMATE_COST`. También se puede habilitar esta directiva en un símbolo del sistema:

```
ora2pg -t SHOW_REPORT --estimate_cost
```

La unidad de migración predeterminada representa aproximadamente cinco minutos para un experto en PostgreSQL. Si esta migración es la primera, puede aumentar la unidad de migración predeterminada mediante la directiva de configuración `COST_UNIT_VALUE` o la opción de línea de comandos `--cost_unit_value`.

En la última línea del informe se muestra el código de migración total estimado en días por persona. La estimación sigue el número de unidades de migración estimadas para cada objeto.

En el ejemplo de código siguiente se ven algunas variaciones de evaluación: 
* Evaluación de tablas
* Evaluación de columnas
* Evaluación de esquemas que usa una unidad de costo predeterminada de 5 minutos
* Evaluación de esquemas que usa una unidad de costo de 10 minutos

```
ora2pg -t SHOW_TABLE -c c:\ora2pg\ora2pg_hr.conf > c:\ts303\hr_migration\reports\tables.txt 
ora2pg -t SHOW_COLUMN -c c:\ora2pg\ora2pg_hr.conf > c:\ts303\hr_migration\reports\columns.txt
ora2pg -t SHOW_REPORT -c c:\ora2pg\ora2pg_hr.conf --dump_as_html --estimate_cost > c:\ts303\hr_migration\reports\report.html
ora2pg -t SHOW_REPORT -c c:\ora2pg\ora2pg_hr.conf –-cost_unit_value 10 --dump_as_html --estimate_cost > c:\ts303\hr_migration\reports\report2.html
```

Esta es la salida del nivel de migración de evaluación de esquemas B-5:

* Niveles de migración:

  * A: Migración que se puede ejecutar automáticamente
    
  * B: Migración con reescritura de código y un costo de hasta 5 días-persona.
    
  * C: Migración con reescritura de código y un costo de más de 5 días por persona
    
* Niveles técnicos:

   * 1 = Trivial: sin funciones almacenadas ni desencadenadores

   * 2 = Fácil: sin funciones almacenadas, pero con desencadenadores; sin reescritura manual

   * 3 = Simple: funciones almacenadas o desencadenadores; sin reescritura manual

   * 4 = Manual: sin funciones almacenadas, pero con desencadenadores o vistas con reescritura de código

   * 5 = Difícil: funciones almacenadas o desencadenadores con reescritura de código

La evaluación consta de: 
* Una letra (A o B) para especificar si la migración necesita reescritura manual.

* Un número de 1 a 5 para indicar la dificultad técnica. 

Otra opción, `-human_days_limit`, especifica el límite de días por persona. En este caso, establezca el nivel de migración en C para indicar que esta necesita una gran cantidad de trabajo, la administración completa del proyecto y soporte de migración. El valor predeterminado es 10 días por persona. Puede usar la directiva de configuración `HUMAN_DAYS_LIMIT` para cambiar este valor predeterminado de manera permanente.

Esta evaluación de esquemas se desarrolló para ayudar a los usuarios a decidir qué base de datos migrar primero y qué equipos movilizar.

### <a name="convert"></a>Convert

En migraciones con un tiempo de inactividad mínimo, el origen de la migración cambia. Se aparta del destino en términos de datos y esquema después de la migración única. Durante la fase de *sincronización de datos*, asegúrese de que todos los cambios en el origen se capturen y se apliquen al destino prácticamente en tiempo real. Después de comprobar que todos los cambios se hayan aplicado al destino, puede *realizar una migración total* del origen al entorno de destino.

En este paso de la migración, el código de Oracle y los scripts DDL se convierten o se traducen a PostgreSQL. La herramienta ora2pg exporta automáticamente los objetos de Oracle en el formato de PostgreSQL. Algunos de los objetos generados no se pueden compilar en la base de datos de PostgreSQL sin cambios manuales.  

Para entender qué elementos necesitan intervención manual, primero compile los archivos generados por ora2pg en la base de datos de PostgreSQL. Compruebe el registro y realice los cambios necesarios hasta que la estructura del esquema sea compatible con la sintaxis de PostgreSQL.


#### <a name="create-a-migration-template"></a>Creación de una plantilla de migración 

Se recomienda usar la plantilla de migración que proporciona ora2pg. Cuando se usan las opciones `--project_base` y `--init_project`, ora2pg crea una plantilla de proyecto con un árbol de trabajo, un archivo de configuración y un script para exportar todos los objetos de la base de datos de Oracle. Para más información, consulte la [documentación de ora2pg](https://ora2pg.darold.net/documentation.html).

Use el comando siguiente: 

```
ora2pg --project_base /app/migration/ --init_project test_project
```

Esta es la salida del ejemplo: 
   
```
ora2pg --project_base /app/migration/ --init_project test_project
        Creating project test_project.
        /app/migration/test_project/
                schema/
                        dblinks/
                        directories/
                        functions/
                        grants/
                        mviews/
                        packages/
                        partitions/
                        procedures/
                        sequences/
                        synonyms/
                        tables/
                        tablespaces/
                        triggers/
                        types/
                        views/
                sources/
                        functions/
                        mviews/
                        packages/
                        partitions/
                        procedures/
                        triggers/
                        types/
                        views/
                data/
                config/
                reports/

        Generating generic configuration file
        Creating script export_schema.sh to automate all exports.
        Creating script import_all.sh to automate all imports.
```

El directorio `sources/` contiene el código de Oracle. El directorio `schema/` contiene el código trasladado a PostgreSQL. El directorio `reports/` contiene los informes HTML y la evaluación del costo de la migración.


Una vez creada la estructura del proyecto, se crea un archivo de configuración genérico. Defina la conexión de base de datos de Oracle y los parámetros de configuración relevantes en el archivo de configuración. Para obtener más información sobre el archivo de configuración, vea la [documentación de ora2pg](https://ora2pg.darold.net/documentation.html).


#### <a name="export-oracle-objects"></a>Exportación de objetos de Oracle

A continuación, exporte los objetos de Oracle como objetos de PostgreSQL. Para ello, ejecute el archivo *export_schema.sh*.

```
cd /app/migration/mig_project
./export_schema.sh
```

Ejecute el siguiente comando de forma manual.

```
SET namespace="/app/migration/mig_project"

ora2pg -p -t DBLINK -o dblink.sql -b %namespace%/schema/dblinks -c %namespace%/config/ora2pg.conf
ora2pg -p -t DIRECTORY -o directory.sql -b %namespace%/schema/directories -c %namespace%/config/ora2pg.conf
ora2pg -p -t FUNCTION -o functions2.sql -b %namespace%/schema/functions -c %namespace%/config/ora2pg.conf 
ora2pg -p -t GRANT -o grants.sql -b %namespace%/schema/grants -c %namespace%/config/ora2pg.conf 
ora2pg -p -t MVIEW -o mview.sql -b %namespace%/schema/mviews -c %namespace%/config/ora2pg.conf
ora2pg -p -t PACKAGE -o packages.sql -b %namespace%/schema/packages -c %namespace%/config/ora2pg.conf
ora2pg -p -t PARTITION -o partitions.sql -b %namespace%/schema/partitions -c %namespace%/config/ora2pg.conf
ora2pg -p -t PROCEDURE -o procs.sql -b %namespace%/schema/procedures -c %namespace%/config/ora2pg.conf
ora2pg -p -t SEQUENCE -o sequences.sql -b %namespace%/schema/sequences -c %namespace%/config/ora2pg.conf
ora2pg -p -t SYNONYM -o synonym.sql -b %namespace%/schema/synonyms -c %namespace%/config/ora2pg.conf
ora2pg -p -t TABLE -o table.sql -b %namespace%/schema/tables -c %namespace%/config/ora2pg.conf 
ora2pg -p -t TABLESPACE -o tablespaces.sql -b %namespace%/schema/tablespaces -c %namespace%/config/ora2pg.conf
ora2pg -p -t TRIGGER -o triggers.sql -b %namespace%/schema/triggers -c %namespace%/config/ora2pg.conf 
ora2pg -p -t TYPE -o types.sql -b %namespace%/schema/types -c %namespace%/config/ora2pg.conf 
ora2pg -p -t VIEW -o views.sql -b %namespace%/schema/views -c %namespace%/config/ora2pg.conf
```

Use el siguiente comando para extraer los datos.

```
ora2pg -t COPY -o data.sql -b %namespace/data -c %namespace/config/ora2pg.conf
```

#### <a name="compile-files"></a>Compilación de archivos

Por último, compile todos los archivos en el servidor de Azure Database for PostgreSQL. Puede optar por cargar los archivos DDL generados de forma manual o por usar el segundo script *import_all.sh* para importar esos archivos de manera interactiva.

```
psql -f %namespace%\schema\sequences\sequence.sql -h server1-server.postgres.database.azure.com -p 5432 -U username@server1-server -d database -l %namespace%\ schema\sequences\create_sequences.log

psql -f %namespace%\schema\tables\table.sql -h server1-server.postgres.database.azure.com p 5432 -U username@server1-server -d database -l %namespace%\schema\tables\create_table.log
```

Este es el comando de importación de datos:

```
psql -f %namespace%\data\table1.sql -h server1-server.postgres.database.azure.com -p 5432 -U username@server1-server -d database -l %namespace%\data\table1.log

psql -f %namespace%\data\table2.sql -h server1-server.postgres.database.azure.com -p 5432 -U username@server1-server -d database -l %namespace%\data\table2.log
```

Mientras se compilan los archivos, compruebe los registros y corrija cualquier sintaxis que ora2pg no pueda convertir por sí mismo.

Para obtener más información, vea [Soluciones alternativas de migración de Oracle a Azure Database for PostgreSQL](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20Database%20for%20PostgreSQL%20Migration%20Workarounds.pdf).

## <a name="migrate"></a>Migrar 

Una vez que tenga los requisitos previos necesarios y haya completado los pasos de premigración, puede iniciar la migración del esquema y los datos.

### <a name="migrate-schema-and-data"></a>Migración de esquemas y datos

Una vez realizadas las correcciones necesarias, hay una compilación estable de la base de datos lista para implementar. Ejecute los comandos de importación `psql` y apunte a los archivos que contienen el código modificado. Esta tarea compila los objetos de base de datos en la base de datos de PostgreSQL e importa los datos.

En este paso se puede implementar un nivel de paralelismo al importar los datos.

### <a name="sync-data-and-cut-over"></a>Sincronización de datos y migración total

En las migraciones en línea (tiempo de inactividad mínimo), el origen de migración sigue cambiando. Se aparta del destino en términos de datos y esquema después de la migración única. 

Durante la fase de *sincronización de datos*, asegúrese de que todos los cambios en el origen se capturen y se apliquen al destino prácticamente en tiempo real. Después de comprobar que todos los cambios se hayan aplicado, puede realizar una migración total del origen al entorno de destino.

Para realizar una migración en línea, póngase en contacto con AskAzureDBforPostgreSQL@service.microsoft.com para obtener soporte técnico.

En una migración *diferencial o incremental* que use ora2pg, en cada tabla emplee una consulta que filtre (*corte*) por fecha, hora u otro parámetro. Luego finalice la migración mediante una segunda consulta que migre los datos restantes.

En la tabla de datos de origen, migre primero todos los datos históricos. Este es un ejemplo:

```
select * from table1 where filter_data < 01/01/2019
```

Puede consultar los cambios desde la migración inicial si ejecuta un comando como este:

```
select * from table1 where filter_data >= 01/01/2019
```

En este caso se recomienda mejorar la validación mediante la comprobación de la paridad de datos en ambos lados, el origen y el destino.

## <a name="post-migration"></a>Etapa posterior a la migración 

Cuando haya completado correctamente la fase de *migración*, realice las tareas posteriores a la migración para asegurarse de que todo funciona de la forma más fluida y eficaz posible.

### <a name="remediate-applications"></a>Corrección de las aplicaciones

Cuando se hayan migrado los datos al entorno de destino, todas las aplicaciones que antes utilizaban el origen deben empezar a utilizar el destino. A veces la configuración requiere cambios en las aplicaciones.

### <a name="test"></a>Prueba

Después de migrar los datos al destino, ejecute pruebas en las bases de datos para comprobar que las aplicaciones funcionan bien con el destino. Para asegurarse de que el origen y el destino se hayan migrado correctamente, ejecute los scripts de validación de datos manuales en las bases de datos de origen de Oracle y de destino de PostgreSQL.

Idealmente, si las bases de datos de origen y destino tienen una ruta de acceso de red, se debe usar ora2pg para la validación de datos. Puede utilizar la acción `TEST` para asegurarse de que todos los objetos de la base de datos de Oracle se han creado en PostgreSQL. 

Ejecute este comando:

```
ora2pg -t TEST -c config/ora2pg.conf > migration_diff.txt
```

### <a name="optimize"></a>Optimización

La fase posterior a la migración es vital para resolver problemas de precisión de los datos y para comprobar su integridad. En esta fase también se abordan los problemas de rendimiento de la carga de trabajo.

## <a name="migration-assets"></a>Recursos de migración 

Para obtener más información sobre este escenario de migración, vea los siguientes recursos. Ayudan con la integración del proyecto de migración real.

| Recurso | Descripción    |
| -------------- | ------------------ |
| [Libro de cocina de la migración de Oracle a Azure PostgreSQL](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20PostgreSQL%20Migration%20Cookbook.pdf) | Este documento ayuda a los arquitectos, consultores, administradores de bases de datos y roles relacionados a migrar rápidamente cargas de trabajo de Oracle a Azure Database for PostgreSQL mediante ora2pg. |
| [Soluciones alternativas de migración de Oracle a Azure PostgreSQL](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20Database%20for%20PostgreSQL%20Migration%20Workarounds.pdf) | Este documento ayuda a los arquitectos, consultores, administradores de bases de datos y roles relacionados a solucionar rápidamente los problemas al migrar cargas de trabajo de Oracle a Azure Database for PostgreSQL. |
| [Pasos para instalar ora2pg en Windows o Linux](https://github.com/microsoft/DataMigrationTeam/blob/master/Whitepapers/Steps%20to%20Install%20ora2pg%20on%20Windows%20and%20Linux.pdf)                       | En este documento se proporciona una guía de instalación rápida para migrar el esquema y los datos de Oracle a Azure Database for PostgreSQL mediante ora2pg en Windows o Linux. Para más información, consulte la [documentación de ora2pg](http://ora2pg.darold.net/documentation.html). |

El equipo de ingeniería de datos SQL ha desarrollado estos recursos. El objetivo principal de este equipo es desbloquear y acelerar la modernización compleja de proyectos de migración de una plataforma de datos a la plataforma de datos de Microsoft Azure.

## <a name="more-support"></a>Más soporte técnico

Para obtener ayuda con la migración más allá del ámbito de las herramientas de ora2pg, póngase en contacto con [@Ask Azure DB for PostgreSQL](mailto:AskAzureDBforPostgreSQL@service.microsoft.com).

## <a name="next-steps"></a>Pasos siguientes

Para obtener una matriz de servicios y herramientas para la migración de datos y bases de datos y para las tareas especializadas, vea [Servicios y herramientas para la migración de datos](../dms/dms-tools-matrix.md).

Documentación: 
- [Documentación de Azure Database for PostgreSQL](./index.yml)
- [Documentación de ora2pg](https://ora2pg.darold.net/documentation.html)
- [Sitio web de PostgreSQL](https://www.postgresql.org/)
- [Compatibilidad con transacciones autónomas en PostgreSQL](http://blog.dalibo.com/2016/08/19/Autonoumous_transactions_support_in_PostgreSQL.html) 