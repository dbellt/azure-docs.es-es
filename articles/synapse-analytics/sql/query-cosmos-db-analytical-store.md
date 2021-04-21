---
title: Consulta de datos de Azure Cosmos DB mediante un grupo de SQL sin servidor en Azure Synapse Link
description: En este artículo, aprenderá a consultar Azure Cosmos DB mediante un grupo de SQL sin servidor en Azure Synapse Link.
services: synapse analytics
author: jovanpop-msft
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 03/02/2021
ms.author: jovanpop
ms.reviewer: jrasnick
ms.custom: cosmos-db
ms.openlocfilehash: 64a112fd29ee9e3fbb82d9b54322415569b3ff85
ms.sourcegitcommit: c3739cb161a6f39a9c3d1666ba5ee946e62a7ac3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107209543"
---
# <a name="query-azure-cosmos-db-data-with-a-serverless-sql-pool-in-azure-synapse-link"></a>Consulta de datos de Azure Cosmos DB con un grupo de SQL sin servidor en Azure Synapse Link

Un grupo de SQL sin servidor permite analizar los datos de los contenedores de Azure Cosmos DB que están habilitados con [Azure Synapse Link](../../cosmos-db/synapse-link.md) casi en tiempo real, sin afectar al rendimiento de las cargas de trabajo transaccionales. Ofrece una sintaxis T-SQL familiar para consultar los datos del [almacén analítico](../../cosmos-db/analytical-store-introduction.md) y la conectividad integrada en una amplia gama de herramientas de consulta ad hoc y de inteligencia empresarial (BI) a través de la interfaz de T-SQL.

Para consultar Azure Cosmos DB, se admite el área expuesta de [SELECT](/sql/t-sql/queries/select-transact-sql?view=azure-sqldw-latest&preserve-view=true) completa mediante la función [OPENROWSET](develop-openrowset.md), que incluye la mayoría de las [funciones y los operadores de SQL](overview-features.md). También puede almacenar los resultados de la consulta que lee los datos de Azure Cosmos DB junto con los de Azure Blob Storage o Azure Data Lake Storage mediante [create external table as select](develop-tables-cetas.md#cetas-in-serverless-sql-pool) (CETAS). Actualmente no se pueden almacenar los resultados de las consultas de un grupo de SQL sin servidor en Azure Cosmos DB mediante CETAS.

En este artículo, aprenderá a escribir una consulta con un grupo de SQL sin servidor que consultará datos de contenedores de Azure Cosmos DB que están habilitados con Azure Synapse Link. Después, puede obtener más información sobre la creación de vistas de un grupo de SQL sin servidor en contenedores de Azure Cosmos DB y cómo conectarlas a modelos de Power BI en [este tutorial](./tutorial-data-analyst.md). En este tutorial se usa un contenedor con un [esquema bien definido de Azure Cosmos DB](../../cosmos-db/analytical-store-introduction.md#schema-representation).

## <a name="overview"></a>Información general

El grupo de SQL sin servidor permite consultar el almacenamiento analítico de Azure Cosmos DB mediante la función `OPENROWSET`. 
- `OPENROWSET` con clave insertada. Esta sintaxis se puede usar para consultar colecciones de Azure Cosmos DB sin necesidad de preparar las credenciales.
- `OPENROWSET` que hizo referencia a las credenciales que contiene la clave de cuenta de Cosmos DB. Esta sintaxis se puede usar para crear vistas en las colecciones de Azure Cosmos DB.

### <a name="openrowset-with-key"></a>[OPENROWSET con clave](#tab/openrowset-key)

Para admitir consultas y análisis de datos en un almacén analítico de Azure Cosmos DB, se usa un grupo de SQL sin servidor. El grupo de SQL sin servidor usa la sintaxis SQL `OPENROWSET`, por lo que la cadena de conexión de Azure Cosmos DB se debe convertir primero a este formato:

```sql
OPENROWSET( 
       'CosmosDB',
       '<SQL connection string for Azure Cosmos DB>',
       <Container name>
    )  [ < with clause > ] AS alias
```

En la cadena de conexión SQL para Azure Cosmos DB se especifican el nombre de cuenta de Azure Cosmos DB, el nombre de la base de datos, la clave maestra de la cuenta de base de datos y un nombre de región opcional para la función `OPENROWSET`. Parte de esta información se puede tomar de la cadena de conexión estándar de Azure Cosmos DB.

Conversión del formato de cadena de conexión estándar de Azure Cosmos DB:

```
AccountEndpoint=https://<database account name>.documents.azure.com:443/;AccountKey=<database account master key>;
```

La cadena de conexión SQL tiene el formato siguiente:

```sql
'account=<database account name>;database=<database name>;region=<region name>;key=<database account master key>'
```

La región es opcional. Si se omite, se usa la región primaria del contenedor.

El nombre del contenedor de Azure Cosmos DB se especifica sin comillas con la sintaxis `OPENROWSET`. Si el nombre del contenedor tiene algún carácter especial, por ejemplo, un guion (-), se debe encapsular dentro de corchetes (`[]`) con la sintaxis `OPENROWSET`.

### <a name="openrowset-with-credential"></a>[OPENROWSET con credenciales](#tab/openrowset-credential)

Puede usar la sintaxis `OPENROWSET` que hace referencia a las credenciales:

```sql
OPENROWSET( 
       PROVIDER = 'CosmosDB',
       CONNECTION = '<SQL connection string for Azure Cosmos DB without account key>',
       OBJECT = '<Container name>',
       [ CREDENTIAL | SERVER_CREDENTIAL ] = '<credential name>'
    )  [ < with clause > ] AS alias
```

En este caso, la cadena de conexión SQL para Azure Cosmos DB no contiene una clave. La cadena de conexión de dispositivo tiene el formato siguiente:

```sql
'account=<database account name>;database=<database name>;region=<region name>'
```

La clave maestra de la cuenta de la base de datos se ubica en las credenciales a nivel de servidor o en las credenciales en el ámbito de la base de datos. 

---

> [!IMPORTANT]
> Asegúrese de usar alguna intercalación de base de datos UTF-8 (por ejemplo, `Latin1_General_100_CI_AS_SC_UTF8`) porque los valores de cadena de un almacén analítico de Azure Cosmos DB se codifican como texto UTF-8.
> Una falta de coincidencia entre la codificación de texto del archivo y la intercalación podría producir errores de conversión de texto inesperados.
> Puede cambiar fácilmente la intercalación predeterminada de la base de datos actual mediante la instrucción T-SQL `alter database current collate Latin1_General_100_CI_AI_SC_UTF8`.

> [!NOTE]
> Un grupo de SQL sin servidor no admite la realización de consultas en un almacén transaccional de Azure Cosmos DB.

## <a name="sample-dataset"></a>Conjunto de datos de ejemplo

Los ejemplos de este artículo se basan en datos de [European Centre for Disease Prevention and Control (ECDC) COVID-19 Cases](https://azure.microsoft.com/services/open-datasets/catalog/ecdc-covid-19-cases/) (Casos de COVID-19 del Centro Europeo para la Prevención y Control de Enfermedades) y [COVID-19 Open Research Dataset (CORD-19), doi:10.5281/zenodo.3715505](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/).

Puede ver la licencia y la estructura de los datos en estas páginas. También puede descargar datos de ejemplo para los conjuntos de datos [ECDC](https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases/latest/ecdc_cases.json) y [CORD-19](https://azureopendatastorage.blob.core.windows.net/covid19temp/comm_use_subset/pdf_json/000b7d1517ceebb34e1e3e817695b6de03e2fa78.json).

Para seguir con este artículo, en el que se muestra cómo consultar datos de Azure Cosmos DB con un grupo de SQL sin servidor, asegúrese de que crea los recursos siguientes:

* Una cuenta de base de datos de Azure Cosmos DB [habilitada para Azure Synapse Link](../../cosmos-db/configure-synapse-link.md)
* Una base de datos de Azure Cosmos DB con el nombre `covid`
* Dos contenedores de Azure Cosmos DB denominados `Ecdc` y `Cord19` cargados con los conjuntos de datos de ejemplo anteriores

Puede usar la cadena de conexión siguiente para fines de pruebas: `Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==`. Tenga en cuenta que esta conexión no garantizará el rendimiento porque esta cuenta podría estar ubicada en una región remota en comparación con el punto de conexión de Synapse SQL.

## <a name="explore-azure-cosmos-db-data-with-automatic-schema-inference"></a>Exploración de datos de Azure Cosmos DB con inferencia automática del esquema

La forma más fácil de explorar datos en Azure Cosmos DB consiste en usar la funcionalidad de inferencia automática del esquema. Al omitir la cláusula `WITH` de la instrucción `OPENROWSET`, puede indicar al grupo de SQL sin servidor que detecte automáticamente (infiera) el esquema del almacén analítico del contenedor de Azure Cosmos DB.

### <a name="openrowset-with-key"></a>[OPENROWSET con clave](#tab/openrowset-key)

```sql
SELECT TOP 10 *
FROM OPENROWSET( 
       'CosmosDB',
       'Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==',
       Ecdc) as documents
```

### <a name="openrowset-with-credential"></a>[OPENROWSET con credenciales](#tab/openrowset-credential)

```sql
/*  Setup - create server-level or database scoped credential with Azure Cosmos DB account key:
    CREATE CREDENTIAL MyCosmosDbAccountCredential
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = 's5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==';
*/
SELECT TOP 10 *
FROM OPENROWSET(
      PROVIDER = 'CosmosDB',
      CONNECTION = 'Account=synapselink-cosmosdb-sqlsample;Database=covid',
      OBJECT = 'Ecdc',
      SERVER_CREDENTIAL = 'MyCosmosDbAccountCredential'
    ) with ( date_rep varchar(20), cases bigint, geo_id varchar(6) ) as rows
```

---

En el ejemplo anterior, se indica a un grupo de SQL sin servidor que se conecte a la base de datos `covid` en la cuenta de Azure Cosmos DB `MyCosmosDbAccount` autenticada con la clave de Azure Cosmos DB (en el ejemplo anterior es ficticia). Después, accedimos al almacén analítico del contenedor `Ecdc` en la región `West US 2`. Como no se proyectan propiedades específicas, la función `OPENROWSET` devolverá todas las propiedades de los elementos de Azure Cosmos DB.

Suponiendo que los elementos del contenedor de Azure Cosmos DB tienen las propiedades `date_rep`, `cases` y `geo_id`, los resultados de esta consulta se muestran en la tabla siguiente:

| date_rep | cases | geo_id |
| --- | --- | --- |
| 2020-08-13 | 254 | RS |
| 2020-08-12 | 235 | RS |
| 2020-08-11 | 163 | RS |

Si tiene que explorar datos del otro contenedor de la misma base de datos de Azure Cosmos DB, puede usar la misma cadena de conexión y hacer referencia al contenedor necesario como tercer parámetro:

```sql
SELECT TOP 10 *
FROM OPENROWSET( 
       'CosmosDB',
       'Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==',
       Cord19) as cord19
```

## <a name="explicitly-specify-schema"></a>Especificación explícita del esquema

Aunque la funcionalidad de inferencia de esquema automática en `OPENROWSET` proporciona una experiencia fácil de usar, es posible que para escenarios empresariales tenga que especificar de forma explícita el esquema para leer solo las propiedades pertinentes de los datos de Azure Cosmos DB.

La función `OPENROWSET` le permite especificar de forma explícita las propiedades que quiere leer de los datos del contenedor, junto a sus tipos de datos.

Imagine que ha importado algunos datos del [conjunto de datos ECDC COVID](https://azure.microsoft.com/services/open-datasets/catalog/ecdc-covid-19-cases/) con la siguiente estructura en Azure Cosmos DB:

```json
{"date_rep":"2020-08-13","cases":254,"countries_and_territories":"Serbia","geo_id":"RS"}
{"date_rep":"2020-08-12","cases":235,"countries_and_territories":"Serbia","geo_id":"RS"}
{"date_rep":"2020-08-11","cases":163,"countries_and_territories":"Serbia","geo_id":"RS"}
```

Estos documentos JSON planos en Azure Cosmos DB se pueden representar como un conjunto de filas y columnas en Synapse SQL. La función `OPENROWSET` le permite especificar un subconjunto de propiedades que quiere leer y los tipos de columna exactos en la cláusula `WITH`:

### <a name="openrowset-with-key"></a>[OPENROWSET con clave](#tab/openrowset-key)

```sql
SELECT TOP 10 *
FROM OPENROWSET(
      'CosmosDB',
      'Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==',
       Ecdc
    ) with ( date_rep varchar(20), cases bigint, geo_id varchar(6) ) as rows
```

### <a name="openrowset-with-credential"></a>[OPENROWSET con credenciales](#tab/openrowset-credential)

```sql
/*  Setup - create server-level or database scoped credential with Azure Cosmos DB account key:
    CREATE CREDENTIAL MyCosmosDbAccountCredential
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = 's5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==';
*/
SELECT TOP 10 *
FROM OPENROWSET(
      PROVIDER = 'CosmosDB',
      CONNECTION = 'Account=synapselink-cosmosdb-sqlsample;Database=covid',
      OBJECT = 'Ecdc',
      SERVER_CREDENTIAL = 'MyCosmosDbAccountCredential'
    ) with ( date_rep varchar(20), cases bigint, geo_id varchar(6) ) as rows
   
```

---
El resultado de esta consulta podría ser similar al de la tabla siguiente:

| date_rep | cases | geo_id |
| --- | --- | --- |
| 2020-08-13 | 254 | RS |
| 2020-08-12 | 235 | RS |
| 2020-08-11 | 163 | RS |

Para obtener más información sobre los tipos de SQL que se deben usar para los valores de Azure Cosmos DB, revise las [reglas de asignaciones de tipos SQL](#azure-cosmos-db-to-sql-type-mappings) al final del artículo.

## <a name="create-view"></a>Creación de vistas

No se recomienda ni se ofrece asistencia para la creación de vistas en las bases de datos maestra o predeterminada. Por lo tanto, debe crear una base de datos de usuario para las vistas.

Una vez que identifique el esquema, puede preparar una vista sobre los datos de Azure Cosmos DB. Debe colocar la clave de cuenta de Azure Cosmos DB en unas credenciales independientes y hacer referencia a ellas a partir de la función `OPENROWSET`. No mantenga su clave de cuenta en la definición de la vista.

```sql
CREATE CREDENTIAL MyCosmosDbAccountCredential
WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = 's5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==';
GO
CREATE OR ALTER VIEW Ecdc
AS SELECT *
FROM OPENROWSET(
      PROVIDER = 'CosmosDB',
      CONNECTION = 'Account=synapselink-cosmosdb-sqlsample;Database=covid',
      OBJECT = 'Ecdc',
      SERVER_CREDENTIAL = 'MyCosmosDbAccountCredential'
    ) with ( date_rep varchar(20), cases bigint, geo_id varchar(6) ) as rows
```

No utilice `OPENROWSET` sin un esquema definido explícitamente, ya que podría afectar al rendimiento. Asegúrese de usar los tamaños más pequeños posibles para las columnas; por ejemplo, VARCHAR(100) en lugar de la opción VARCHAR(8000) predeterminada. Debe usar una intercalación UTF-8 como intercalación de base de datos predeterminada o establecerla como intercalación de columna explícita para evitar un [problema de conversión UTF-8](../troubleshoot/reading-utf8-text.md). La intercalación `Latin1_General_100_BIN2_UTF8` proporciona el mejor rendimiento cuando se filtran los datos mediante algunas columnas de cadena.

## <a name="query-nested-objects-and-arrays"></a>Consulta de objetos y matrices anidados

Con Azure Cosmos DB, puede representar modelos de datos más complejos si se crean como matrices u objetos anidados. La funcionalidad de sincronización automática de Azure Synapse Link para Azure Cosmos DB administra la representación del esquema en el almacén analítico de forma integrada, lo que incluye el control de los tipos de datos anidados que permiten realizar consultas enriquecidas desde el grupo de SQL sin servidor.

Por ejemplo, el conjunto de datos [CORD-19](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/) tiene documentos JSON con esta estructura:

```json
{
    "paper_id": <str>,                   # 40-character sha1 of the PDF
    "metadata": {
        "title": <str>,
        "authors": <array of objects>    # list of author dicts, in order
        ...
     }
     ...
}
```

Los objetos y las matrices anidados de Azure Cosmos DB se representan como cadenas JSON en el resultado de la consulta cuando los lee la función `OPENROWSET`. Puede especificar las rutas de acceso a los valores anidados en los objetos cuando use la cláusula `WITH`:

```sql
SELECT TOP 10 *
FROM OPENROWSET( 
       'CosmosDB',
       'Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==',
       Cord19)
WITH (  paper_id    varchar(8000),
        title        varchar(1000) '$.metadata.title',
        metadata     varchar(max),
        authors      varchar(max) '$.metadata.authors'
) AS docs;
```

El resultado de esta consulta podría ser similar al de la tabla siguiente:

| paper_id | title | metadata | authors |
| --- | --- | --- | --- |
| bb11206963e831f… | Información complementaria Un estudio de epide… | `{"title":"Supplementary Informati…` | `[{"first":"Julien","last":"Mélade","suffix":"","af…`| 
| bb1206963e831f1… | The Use of Convalescent Sera in Immune-E… | `{"title":"The Use of Convalescent…` | `[{"first":"Antonio","last":"Lavazza","suffix":"", …` |
| bb378eca9aac649… | Tylosema esculentum (Marama) Tuber and B… | `{"title":"Tylosema esculentum (Ma…` | `[{"first":"Walter","last":"Chingwaru","suffix":"",…` | 

Obtenga más información sobre cómo analizar [tipos de datos complejos en Azure Synapse Link](../how-to-analyze-complex-schema.md) y [estructuras anidadas en un grupo de SQL sin servidor](query-parquet-nested-types.md).

> [!IMPORTANT]
> Si ve caracteres inesperados en el texto, como `MÃƒÂ©lade` en lugar de `Mélade`, la intercalación de la base de datos no está establecida en [UTF-8](/sql/relational-databases/collations/collation-and-unicode-support#utf8).
> [Cambie la intercalación de la base de datos](/sql/relational-databases/collations/set-or-change-the-database-collation#to-change-the-database-collation) por una intercalación UTF-8 mediante una instrucción SQL como `ALTER DATABASE MyLdw COLLATE LATIN1_GENERAL_100_CI_AS_SC_UTF8`.

## <a name="flatten-nested-arrays"></a>Acoplamiento de matrices anidadas

Los datos de Azure Cosmos DB pueden tener submatrices anidadas como la matriz del creador de un conjunto de datos [CORD-19](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/):

```json
{
    "paper_id": <str>,                      # 40-character sha1 of the PDF
    "metadata": {
        "title": <str>,
        "authors": [                        # list of author dicts, in order
            {
                "first": <str>,
                "middle": <list of str>,
                "last": <str>,
                "suffix": <str>,
                "affiliation": <dict>,
                "email": <str>
            },
            ...
        ],
        ...
}
```

En algunos casos, puede que tenga que "combinar" las propiedades del elemento superior (metadata) con todos los elementos de la matriz (authors). Un grupo de SQL sin servidor permite acoplar estructuras anidadas mediante la aplicación de la función `OPENJSON` en la matriz anidada:

```sql
SELECT
    *
FROM
    OPENROWSET(
      'CosmosDB',
      'Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==',
       Cord19
    ) WITH ( title varchar(1000) '$.metadata.title',
             authors varchar(max) '$.metadata.authors' ) AS docs
      CROSS APPLY OPENJSON ( authors )
                  WITH (
                       first varchar(50),
                       last varchar(50),
                       affiliation nvarchar(max) as json
                  ) AS a
```

El resultado de esta consulta podría ser similar al de la tabla siguiente:

| title | authors | first | last | affiliation |
| --- | --- | --- | --- | --- |
| Información complementaria Un estudio de epide… |   `[{"first":"Julien","last":"Mélade","suffix":"","affiliation":{"laboratory":"Centre de Recher…` | Julien | Mélade | `   {"laboratory":"Centre de Recher…` |
Información complementaria Un estudio de epide… | `[{"first":"Nicolas","last":"4#","suffix":"","affiliation":{"laboratory":"","institution":"U…` | Nicolas | N.° 4 |`{"laboratory":"","institution":"U…` | 
| Información complementaria Un estudio de epide… |   `[{"first":"Beza","last":"Ramazindrazana","suffix":"","affiliation":{"laboratory":"Centre de Recher…` | Beza | Ramazindrazana | `{"laboratory":"Centre de Recher…` |
| Información complementaria Un estudio de epide… |   `[{"first":"Olivier","last":"Flores","suffix":"","affiliation":{"laboratory":"UMR C53 CIRAD, …` | Olivier | Flores |`{"laboratory":"UMR C53 CIRAD, …` |     

> [!IMPORTANT]
> Si ve caracteres inesperados en el texto, como `MÃƒÂ©lade` en lugar de `Mélade`, la intercalación de la base de datos no está establecida en [UTF-8](/sql/relational-databases/collations/collation-and-unicode-support#utf8). [Cambie la intercalación de la base de datos](/sql/relational-databases/collations/set-or-change-the-database-collation#to-change-the-database-collation) por una intercalación UTF-8 mediante una instrucción SQL como `ALTER DATABASE MyLdw COLLATE LATIN1_GENERAL_100_CI_AS_SC_UTF8`.

## <a name="azure-cosmos-db-to-sql-type-mappings"></a>Asignaciones de Azure Cosmos DB a tipos de SQL

Aunque el almacén transaccional de Azure Cosmos DB es independiente del esquema, el almacén analítico está esquematizado para optimizar el rendimiento de las consultas analíticas. Con la funcionalidad de sincronización automática de Azure Synapse Link, Azure Cosmos DB administra la representación del esquema en el almacén analítico de forma integrada, lo que incluye el control de los tipos de datos anidados. Dado que un grupo de SQL sin servidor consulta el almacén analítico, es importante comprender cómo asignar tipos de datos de entrada de Azure Cosmos DB a tipos de datos SQL.

Las cuentas de Azure Cosmos DB de SQL (Core) API admiten tipos de propiedad JSON de número, cadena, booleano, nulo, objeto anidado o matriz. Tendrá que elegir tipos SQL que coincidan con estos tipos JSON si usa la cláusula `WITH` en `OPENROWSET`. En la tabla siguiente se muestran los tipos de columna SQL que se deben usar para los distintos tipos de propiedad en Azure Cosmos DB.

| Tipo de propiedad de Azure Cosmos DB | Tipo de columna SQL |
| --- | --- |
| Boolean | bit |
| Entero | bigint |
| Decimal | FLOAT |
| String | varchar (intercalación de base de datos UTF-8) |
| Fecha y hora (cadena con formato ISO) | varchar(30) |
| Fecha y hora (marca de tiempo de UNIX) | bigint |
| Null | `any SQL type` 
| Objeto o matriz anidados | varchar(max) (intercalación de base de datos UTF-8), serializado como texto JSON |

## <a name="full-fidelity-schema"></a>Esquema de fidelidad completa

El esquema de fidelidad completa de Azure Cosmos DB registra los valores y sus tipos de mejor coincidencia para cada propiedad de un contenedor. La función `OPENROWSET` en un contenedor con un esquema de fidelidad completa proporciona el tipo y el valor real en cada celda. Imagine que la consulta siguiente lee los elementos de un contenedor con un esquema de fidelidad completa:

```sql
SELECT *
FROM OPENROWSET(
      'CosmosDB',
      'account=MyCosmosDbAccount;database=covid;region=westus2;key=C0Sm0sDbKey==',
       Ecdc
    ) as rows
```

El resultado de esta consulta devolverá tipos y valores con formato de texto JSON:

| date_rep | cases | geo_id |
| --- | --- | --- |
| {"date":"2020-08-13"} | {"int32":"254"} | {"string":"RS"} |
| {"date":"2020-08-12"} | {"int32":"235"}| {"string":"RS"} |
| {"date":"2020-08-11"} | {"int32":"316"} | {"string":"RS"} |
| {"date":"2020-08-10"} | {"int32":"281"} | {"string":"RS"} |
| {"date":"2020-08-09"} | {"int32":"295"} | {"string":"RS"} |
| {"string":"2020/08/08"} | {"int32":"312"} | {"string":"RS"} |
| {"date":"2020-08-07"} | {"float64":"339.0"} | {"string":"RS"} |

En cada valor, puede ver el tipo identificado en un elemento del contenedor de Azure Cosmos DB. La mayoría de los valores de la propiedad `date_rep` contienen valores `date`, pero algunos de ellos se almacenan incorrectamente como cadenas en Azure Cosmos DB. El esquema de fidelidad completa devolverá valores `date` con el tipo correcto y valores `string` con formato incorrecto.
El número de casos es información almacenada como un valor `int32`, pero hay un valor que se especifica como un número decimal. Este valor tiene el tipo `float64`. Si hay algunos valores que superan el número `int32` mayor, se almacenarían como el tipo `int64`. Todos los valores `geo_id` de este ejemplo se almacenan como tipos `string`.

> [!IMPORTANT]
> La función `OPENROWSET` sin una cláusula `WITH` expone valores con los tipos esperados y valores con tipos especificados incorrectamente. Esta función está diseñada para la exploración de datos y no para la creación de informes. No analice los valores JSON devueltos por esta función para elaborar informes. Use una [cláusula WITH](#query-items-with-full-fidelity-schema) explícita para crear los informes. Debe limpiar los valores que tienen tipos incorrectos en el contenedor de Azure Cosmos DB para aplicar correcciones en el almacén analítico de fidelidad completa.

Si necesita consultar cuentas de Azure Cosmos DB del tipo Mongo DB API, puede obtener más información sobre la representación de esquemas de fidelidad completa en el almacén analítico y los nombres de propiedad extendida que se van a usar en [¿Qué es el almacén analítico de Azure Cosmos DB?](../../cosmos-db/analytical-store-introduction.md#analytical-schema)

### <a name="query-items-with-full-fidelity-schema"></a>Consulta de elementos con un esquema de fidelidad completa

Al consultar el esquema de fidelidad completa, debe especificar de forma explícita el tipo SQL y el tipo de propiedad de Azure Cosmos DB que se espera en la cláusula `WITH`.

En el ejemplo siguiente, se supone que `string` es el tipo correcto para la propiedad `geo_id` y `int32` es el tipo correcto para la propiedad `cases`:

```sql
SELECT geo_id, cases = SUM(cases)
FROM OPENROWSET(
      'CosmosDB'
      'account=MyCosmosDbAccount;database=covid;region=westus2;key=C0Sm0sDbKey==',
       Ecdc
    ) WITH ( geo_id VARCHAR(50) '$.geo_id.string',
             cases INT '$.cases.int32'
    ) as rows
GROUP BY geo_id
```

Los valores para `geo_id` y `cases` que tienen otros tipos se devolverán como valores `NULL`. Esta consulta solo hará referencia a `cases` con el tipo especificado en la expresión (`cases.int32`).

Si tiene valores con otros tipos (`cases.int64`, `cases.float64`) que no se pueden limpiar en un contenedor de Azure Cosmos DB, tendrá que hacerles referencia de forma explícita en una cláusula `WITH` y combinar los resultados. La consulta siguiente agrega los elementos `int32`, `int64` y `float64` almacenados en la columna `cases`:

```sql
SELECT geo_id, cases = SUM(cases_int) + SUM(cases_bigint) + SUM(cases_float)
FROM OPENROWSET(
      'CosmosDB',
      'account=MyCosmosDbAccount;database=covid;region=westus2;key=C0Sm0sDbKey==',
       Ecdc
    ) WITH ( geo_id VARCHAR(50) '$.geo_id.string', 
             cases_int INT '$.cases.int32',
             cases_bigint BIGINT '$.cases.int64',
             cases_float FLOAT '$.cases.float64'
    ) as rows
GROUP BY geo_id
```

En este ejemplo, el número de casos se almacena como valores `int32`, `int64` o `float64`. Todos los valores deben extraerse para calcular el número de casos por país.

## <a name="known-issues"></a>Problemas conocidos

- Un grupo de SQL sin servidor devolverá una advertencia en tiempo de compilación si la intercalación de columnas `OPENROWSET` no tiene codificación UTF-8. Puede cambiar fácilmente la intercalación predeterminada de todas las funciones `OPENROWSET` que se ejecutan en la base de datos actual mediante la instrucción T-SQL `alter database current collate Latin1_General_100_CI_AS_SC_UTF8`.

En la tabla siguiente se enumeran los posibles errores y las acciones para solucionar problemas.

| Error | Causa principal |
| --- | --- |
| Errores de sintaxis:<br/> - Sintaxis incorrecta cerca de `Openrowset`<br/> - `...` no es una opción de proveedor `BULK OPENROWSET` reconocida.<br/> - Sintaxis incorrecta cerca de `...` | Posibles causas principales:<br/> - No se usa CosmosDB como el primer parámetro.<br/> - Se usa un literal de cadena, en lugar de un identificador en el tercer parámetro.<br/> - No se especifica el tercer parámetro (nombre del contenedor). |
| Se produjo un error en la cadena de conexión de CosmosDB. | - No se ha especificado la cuenta, la base de datos ni la clave. <br/> - Hay una opción en una cadena de conexión que no se reconoce.<br/> - Un signo de punto y coma (`;`) se coloca al final de la cadena de conexión. |
| Error al resolver la ruta de acceso de CosmosDB: "Nombre de cuenta incorrecto" o "Nombre de base de datos incorrecto". | No se encuentra el nombre de cuenta, el nombre de la base de datos o el contenedor especificados, o bien no se han habilitado el almacenamiento analítico para la colección especificada.|
| Error al resolver la ruta de acceso de CosmosDB: "Valor de secreto incorrecto" o "El secreto es nulo o está vacío". | La clave de cuenta falta o no es válida. |
| La columna `column name` del tipo `type name` no es compatible con el tipo de datos externo `type name`. | El tipo de columna especificado en la cláusula `WITH` no coincide con el tipo del contenedor de Azure Cosmos DB. Intente cambiar el tipo de columna como se describe en la sección [Asignaciones de Azure Cosmos DB a tipos de SQL](#azure-cosmos-db-to-sql-type-mappings) o use el tipo `VARCHAR`. |
| La columna contiene valores `NULL` en todas las celdas. | Es posible que el nombre de columna o la expresión de la ruta de acceso sean incorrectos en la cláusula `WITH`. El nombre de columna, o la expresión de la ruta de acceso después del tipo de columna, de la cláusula `WITH` debe coincidir con el nombre de alguna propiedad de la colección de Azure Cosmos DB. La comparación *distingue mayúsculas de minúsculas*. Por ejemplo, `productCode` y `ProductCode` son propiedades diferentes. |

Puede informar sugerencias y problemas en la [página de comentarios de Azure Synapse Analytics](https://feedback.azure.com/forums/307516-azure-synapse-analytics?category_id=387862).

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los siguientes artículos.

- [Uso de Power BI y el grupo de SQL sin servidor con Azure Synapse Link](../../cosmos-db/synapse-link-power-bi.md)
- [Creación y uso de vistas en el grupo de SQL sin servidor](create-use-views.md)
- [Tutorial sobre la creación de vistas del grupo de SQL sin servidor en Azure Cosmos DB y su conexión a modelos de Power BI a través de DirectQuery](./tutorial-data-analyst.md)
