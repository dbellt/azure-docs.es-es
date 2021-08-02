---
title: ¿Qué es el almacén analítico de Azure Cosmos DB?
description: Obtenga información sobre el almacén transaccional (basado en filas) y analítico (basado en columnas) de Azure Cosmos DB. Ventajas del almacén analítico, el impacto en el rendimiento de las cargas de trabajo a gran escala y la sincronización automática de datos desde el almacén transaccional al almacén analítico
author: Rodrigossz
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 04/12/2021
ms.author: rosouz
ms.custom: seo-nov-2020
ms.openlocfilehash: 9328b8159b04d4e7e7bc2383739c86c76dbf156a
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111985893"
---
# <a name="what-is-azure-cosmos-db-analytical-store"></a>¿Qué es el almacén analítico de Azure Cosmos DB?
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

El almacén analítico de Azure Cosmos DB es un almacén de columnas completamente aislado para habilitar el análisis a gran escala en los datos operativos de su instancia de Azure Cosmos DB, sin que ello afecte a las cargas de trabajo transaccionales. 

El almacén transaccional de Azure Cosmos DB es independiente del esquema y le permite iterar las aplicaciones transaccionales sin tener que encargarse de la administración de esquemas o índices. A diferencia de esto, el almacén analítico de Azure Cosmos DB está esquematizado para optimizar el rendimiento de las consultas analíticas. En este artículo se describe detalladamente el almacenamiento analítico.

## <a name="challenges-with-large-scale-analytics-on-operational-data"></a>Desafíos del análisis a gran escala de los datos operativos

Los datos operativos de varios modelos de un contenedor de Azure Cosmos DB se almacenan internamente en un "almacén transaccional" basado en filas indexado. El formato del almacén de filas está diseñado para permitir lecturas y escrituras transaccionales rápidas, con tiempos de respuesta en el orden de milisegundos, y consultas operativas. Si el conjunto de datos crece, las consultas analíticas complejas pueden ser costosas en términos de rendimiento aprovisionado en los datos almacenados en este formato. El alto consumo de rendimiento aprovisionado, a su vez, afecta al rendimiento de las cargas de trabajo transaccionales utilizadas por las aplicaciones y los servicios en tiempo real.

Tradicionalmente, para analizar grandes cantidades de datos, los datos operativos se extraen del almacén transaccional de Azure Cosmos DB y se almacenan en una capa de datos independiente. Por ejemplo, los datos se almacenan en un almacenamiento de datos o en un lago de datos con un formato adecuado. Más tarde, estos datos se usan para análisis a gran escala, y se analizan mediante un motor de proceso, como los clústeres de Apache Spark. Esta separación entre las capas de almacenamiento analítico y de proceso y los datos operativos produce latencia adicional, ya que las canalizaciones ETL (extracción, transformación y carga) se ejecutan con menos frecuencia para minimizar el posible impacto en las cargas de trabajo transaccionales.

Las canalizaciones ETL también se vuelven complejas al administrar las actualizaciones de los datos operativos en comparación con cuando se administran únicamente los datos operativos recién ingeridos. 

## <a name="column-oriented-analytical-store"></a>Almacén analítico orientado a columnas

El almacén analítico de Azure Cosmos DB aborda los desafíos de complejidad y latencia que se presentan con las canalizaciones ETL tradicionales. El almacén analítico de Azure Cosmos DB puede sincronizar automáticamente los datos operativos en un almacén de columnas independiente. El formato del almacén de columnas es adecuado para las consultas analíticas a gran escala que se van a realizar de forma optimizada, lo que aumenta la latencia de estas.

Con Azure Synapse Link, ahora puede compilar soluciones de HTAP sin ETL mediante la vinculación directa al almacén analítico de Azure Cosmos DB desde Azure Synapse Analytics. Esto le permite ejecutar análisis a gran escala casi en tiempo real con los datos operativos.

## <a name="features-of-analytical-store"></a>Características del almacén analítico 

Cuando se habilita el almacén analítico en un contenedor de Azure Cosmos DB, se crea internamente un nuevo almacén de columnas en función de los datos operativos del contenedor. Este almacén de columnas se conserva de forma independiente del almacén transaccional orientado a filas de ese contenedor. Las inserciones, actualizaciones y eliminaciones de los datos operativos se sincronizan automáticamente con el almacén analítico. No necesita la fuente de cambios ni ETL para sincronizar los datos.

## <a name="column-store-for-analytical-workloads-on-operational-data"></a>Almacén de columnas para cargas de trabajo analíticas en datos operativos

En general, las cargas de trabajo analíticas implican agregaciones y exámenes secuenciales de los campos seleccionados. Al almacenar los datos en un orden de columna principal, el almacén analítico permite serializar en conjunto un grupo de valores para cada campo. Este formato reduce las IOPS necesarias para examinar o procesar las estadísticas de los campos específicos. Esto mejora drásticamente los tiempos de respuesta de las consultas para los exámenes en grandes conjuntos de datos. 

Por ejemplo, si las tablas operativas se encuentran en el formato siguiente:

:::image type="content" source="./media/analytical-store-introduction/sample-operational-data-table.png" alt-text="Tabla operativa de ejemplo" border="false":::

El almacén de filas conserva los datos anteriores en un formato serializado, por fila, en el disco. Este formato permite lecturas y escrituras transaccionales y consultas operativas más rápidas, como "Devolver información sobre Product1". Sin embargo, a medida que el conjunto de datos crece en gran medida y si desea ejecutar consultas analíticas complejas con los datos, esto puede ser caro. Por ejemplo, si desea obtener "las tendencias de ventas de un producto en la categoría denominada 'Equipment' en diferentes unidades de negocio y meses", debe ejecutar una consulta compleja. Los análisis de gran tamaño de este conjunto de datos pueden resultar caros en cuanto al rendimiento aprovisionado, además de poder afectar al rendimiento de las cargas de trabajo transaccionales que potencian sus aplicaciones y servicios en tiempo real.

El almacén analítico, que es un almacén de columnas, es más adecuado para estas consultas, ya que serializa en conjunto los campos de datos similares y reduce las IOPS del disco.

En la imagen siguiente se muestra el almacén de filas transaccional frente el almacén de columnas analítico en Azure Cosmos DB:

:::image type="content" source="./media/analytical-store-introduction/transactional-analytical-data-stores.png" alt-text="Almacén de filas transaccional frente al almacén de columnas analítico en Azure Cosmos DB" border="false":::

## <a name="decoupled-performance-for-analytical-workloads"></a>Rendimiento desacoplado para cargas de trabajo analíticas

No hay ningún impacto en el rendimiento de las cargas de trabajo transaccionales debido a las consultas analíticas, ya que el almacén analítico es independiente del almacén transaccional.  El almacén analítico no necesita que se asignen Unidades de solicitud (RU) independientes.

## <a name="auto-sync"></a>Sincronización automática

La sincronización automática hace referencia a la funcionalidad totalmente administrada de Azure Cosmos DB donde las inserciones, las actualizaciones y las eliminaciones de datos operativos se sincronizan automáticamente entre el almacén transaccional y el almacén analítico casi en tiempo real. La latencia de sincronización automática suele ser de menos de 2 minutos. En los casos en los que una base de datos de rendimiento compartida cuente con un gran número de contenedores, la latencia de sincronización automática de contenedores individuales puede ser mayor y tardar hasta 5 minutos. Nos gustaría obtener más información sobre cómo encaja esta latencia en sus escenarios. Para ello, póngase en contacto con el [equipo de Azure Cosmos DB](mailto:cosmosdbsynapselink@microsoft.com).

Al final de cada ejecución del proceso de sincronización automática, los datos transaccionales estarán disponibles inmediatamente para los entornos de ejecución de Azure Synapse Analytics:

* Los grupos de Spark de Azure Synapse Analytics pueden leer todos los datos, incluidas las actualizaciones más recientes, mediante tablas de Spark, que se actualizan automáticamente, o mediante el comando `spark.read`, que siempre lee el último estado de los datos.

*  Los grupos sin servidor SQL de Azure Synapse Analytics pueden leer todos los datos, incluidas las actualizaciones más recientes, mediante las vistas, que se actualizan automáticamente, o por medio del comando `SELECT` junto con ` OPENROWSET`, que siempre lee el estado más reciente de los datos.

> [!NOTE]
> Los datos transaccionales se sincronizarán con el almacén analítico incluso si el TTL transaccional es inferior a 2 minutos. 

## <a name="scalability--elasticity"></a>Escalabilidad y elasticidad

Mediante la creación de partición horizontal, el almacén transaccional de Azure Cosmos DB puede escalar elásticamente el almacenamiento y el rendimiento sin tiempo de inactividad. La creación de partición horizontal en el almacén transaccional proporciona escalabilidad y elasticidad en la sincronización automática para garantizar que los datos se sincronicen con el almacén analítico casi en tiempo real. La sincronización de datos se produce independientemente del rendimiento del tráfico transaccional, ya sean 1000 operaciones/s o 1 millón de operaciones/s, y no afecta al rendimiento aprovisionado en el almacén transaccional. 

## <a name="automatically-handle-schema-updates"></a><a id="analytical-schema"></a>Control automático de las actualizaciones de esquema

El almacén transaccional de Azure Cosmos DB es independiente del esquema y le permite iterar las aplicaciones transaccionales sin tener que encargarse de la administración de esquemas o índices. A diferencia de esto, el almacén analítico de Azure Cosmos DB está esquematizado para optimizar el rendimiento de las consultas analíticas. Con la funcionalidad de sincronización automática, Azure Cosmos DB administra la inferencia de esquemas en las actualizaciones más recientes del almacén transaccional. También administra la representación del esquema en el almacén analítico de manera integrada, lo que incluye el control de los tipos de datos anidados.

A medida que evoluciona el esquema, y se agregan propiedades nuevas con el tiempo, el almacén analítico presenta automáticamente un esquema unificado de todos los esquemas históricos del almacén de transacciones.

> [!NOTE]
> En el contexto del almacén analítico, consideramos las siguientes estructuras como propiedad:
> * "Elementos" JSON" o "pares cadena-valor separados por `:`".
> * Objetos JSON, delimitados por `{` y `}`.
> * Matrices JSON, delimitadas por `[` y `]`.


### <a name="schema-constraints"></a>Restricciones del esquema

Las restricciones siguientes se aplican a los datos operativos de Azure Cosmos DB al habilitar el almacén analítico para que realice la inferencia automáticamente y represente el esquema correctamente:

* Puede tener 1000 propiedades como máximo en cualquier nivel de anidamiento del esquema y una profundidad de anidamiento máxima de 127.
  * En el almacén analítico solo se representan las primeras 1000 propiedades.
  * En el almacén analítico solo se representan los primeros 127 niveles anidados.
  * El primer nivel de un documento JSON es su nivel raíz `/`.
  * Las propiedades del primer nivel del documento se representarán como columnas.


* Escenarios de ejemplo:
  * Si el primer nivel del documento tiene 2000 propiedades, solo se representarán las primeras 1000.
  * Si los documentos tienen 5 niveles con 200 propiedades en cada uno, se representarán todas las propiedades.
  * Si los documentos tienen 10 niveles con 400 propiedades en cada uno, solo los dos primeros niveles se representarán completamente en el almacén analítico. También se representará la mitad del tercer nivel.

* El documento hipotético siguiente contiene 4 propiedades y 3 niveles.
  * Los niveles son `root` `myArray` y la estructura anidada dentro de `myArray`.
  * Las propiedades son `id`, `myArray`, `myArray.nested1` y `myArray.nested2`.
  * La representación del almacén analítico tendrá 2 columnas, `id` y `myArray`. Puede usar las funciones de Spark o T-SQL para exponer también las estructuras anidadas como columnas.


```json
{
  "id": "1",
  "myArray": [
    "string1",
    "string2",
    {
      "nested1": "abc",
      "nested2": "cde"
    }
  ]
}
```

* Mientras que los documentos JSON (y colecciones o contenedores de Cosmos DB) distinguen mayúsculas de minúsculas de la perspectiva de exclusividad, el almacén analítico no lo es.

  * **En el mismo documento:** los nombres de propiedades del mismo nivel deben ser únicos cuando se comparan mayúsculas y minúsculas. Por ejemplo, el siguiente documento JSON tiene "Name" y "name" en el mismo nivel. Aunque es un documento JSON válido, no satisface la restricción de exclusividad y, por lo tanto, no se representará por completo en el almacén analítico. En este ejemplo, "Name" y "name" son iguales al compararlos sin hacer distinción de mayúsculas y minúsculas. Solo `"Name": "fred"` se representará en el almacén analítico, ya que es el primer resultado. Asimismo, `"name": "john"` no se representarán en absoluto.
  
  
  ```json
  {"id": 1, "Name": "fred", "name": "john"}
  ```
  
  * **En documentos diferentes:** las propiedades del mismo nivel y con el mismo nombre, pero en casos diferentes, se representarán dentro de la misma columna, con el formato de nombre de la primera aparición. Por ejemplo, los siguientes documentos JSON tienen `"Name"` y `"name"` en el mismo nivel. Dado que el primer formato de documento es `"Name"`, esto es lo que se usará para representar el nombre de la propiedad en el almacén analítico. En otras palabras, el nombre de columna del almacén analítico será `"Name"`. Tanto `"fred"` como `"john"` se representarán en la columna `"Name"`.


  ```json
  {"id": 1, "Name": "fred"}
  {"id": 2, "name": "john"}
  ```


* El primer documento de la colección define el esquema de almacenamiento analítico inicial.
  * Los documentos que tengan más propiedades que el esquema inicial generarán nuevas columnas en el almacén analítico.
  * No se pueden quitar las columnas.
  * La eliminación de todos los documentos de una colección no restablece el esquema del almacén analítico.
  * No hay control de versiones de esquema. La última versión inferida del almacén de transacciones es lo que verá en el almacén analítico.

* Actualmente no se admiten los nombres de la columna de lectura de Azure Synapse Spark que contengan espacios en blanco. Deberá usar funciones de Spark como `cast` o `replace` para poder cargar los datos en un dataframe de Spark.

### <a name="schema-representation"></a>Representación del esquema

En el almacén analítico hay dos maneras de representar el esquema. Estos modos presentan ventajas e inconvenientes en relación con la simplicidad de la representación en columnas, el control de los esquemas polimórficos y la simplicidad de la experiencia de consulta:

* Representación de esquemas bien definida
* Representación de esquemas con fidelidad total

> [!NOTE]
> En el caso de las cuentas de SQL (Core) API, cuando se habilita el almacén analítico, la representación de esquemas predeterminada en él es la representación bien definida. Mientras que para las cuentas de Azure Cosmos DB API para MongoDB, la representación de esquemas predeterminada en el almacén analítico es la representación con fidelidad total. 

**Representación de esquemas bien definida**

La representación de esquemas bien definida crea una representación tabular simple de los datos independientes del esquema en el almacén transaccional. La representación de esquemas bien definida tiene las siguientes características:

* Las propiedades siempre tienen el mismo tipo en los distintos elementos.
* Solo se permite un cambio de tipo 1, de NULL a cualquier otro tipo de datos. La primera aparición que no es NULL define el tipo de datos de columna.

  * Por ejemplo, `{"a":123} {"a": "str"}` no tiene un esquema bien definido porque `"a"` a veces es una cadena y, a veces, un número. En este caso, el almacén analítico registra el tipo de datos de `"a"` como tipo de datos de `“a”` en el primer elemento durante la vigencia del contenedor. Aún así, el documento todavía se incluirá en el almacén analítico, pero los elementos cuyo tipo de datos `"a"` sea distinto no lo serán.
  
    Esta condición no se aplica a las propiedades NULL. Por ejemplo, `{"a":123} {"a":null}` sigue estando bien definida.

* Los tipos de las matrices deben contener un único tipo repetido.

  * Por ejemplo, `{"a": ["str",12]}` no es un esquema bien definido, porque la matriz contiene una mezcla de tipos enteros y de cadenas.

> [!NOTE]
> Si el almacén analítico de Azure Cosmos DB sigue la representación de esquemas bien definida y no se cumplen las especificaciones anteriores en determinados elementos, estos no se incluirán en el almacén analítico.

* Se espera un comportamiento diferente en lo que respecta a los diferentes tipos en el esquema bien definido:
  * Los grupos de Spark en Azure Synapse representarán estos valores como `undefined`.
  * Los grupos sin servidor de SQL en Azure Synapse representarán estos valores como `NULL`.

* Se espera un comportamiento diferente con respecto a los valores explícitos `null`:
  * Los grupos de Spark en Azure Synapse leerán estos valores como `0` (cero). Y cambiará a `undefined` en cuanto la columna tenga un valor distinto de NULL.
  * Los grupos sin servidor SQL de Azure Synapse leerán estos valores como `NULL`.
    
* Se espera un comportamiento diferente con respecto a las columnas que faltan:
  * Los grupos de Spark en Azure Synapse representarán estas columnas como `undefined`.
  * Los grupos sin servidor de SQL en Azure Synapse representarán estas columnas como `NULL`.


**Representación de esquemas con fidelidad total**

La representación de esquemas con fidelidad total está diseñada para administrar todos los esquemas polimórficos de los datos operativos independientes del esquema. En esta representación de esquemas no se quita ningún elemento del almacén analítico aunque no se cumplan las restricciones de los esquemas bien definidos (es decir, que no haya campos ni matrices de tipos de datos mixtos).

Para ello, se traducen las propiedades de hoja de los datos operativos del almacén analítico con columnas distintas según el tipo de datos de los valores de la propiedad. Los nombres de las propiedades de hoja se amplían con tipos de datos como sufijos en el esquema del almacén analítico, de modo que se pueden consultar sin ambigüedad.

En la representación de esquema de fidelidad completa, cada tipo de datos de cada propiedad generará una columna para ese tipo de datos. Cada una de ellas cuenta como una de las 1000 propiedades máximas.

Por ejemplo, vamos a tomar el siguiente documento de ejemplo del almacén transaccional:

```json
{
name: "John Doe",
age: 32,
profession: "Doctor",
address: {
  streetNo: 15850,
  streetName: "NE 40th St.",
  zip: 98052
},
salary: 1000000
}
```

La propiedad de hoja `streetNo` del objeto anidado `address` se representará en el esquema del almacén analítico como columna `address.object.streetNo.int32`. El tipo de datos se agrega como sufijo a la columna. De este modo, si se agrega otro documento al almacén transaccional donde el valor de la propiedad de hoja `streetNo` sea "123" (una cadena), el esquema del almacén analítico evoluciona automáticamente sin modificar el tipo de la columna escrita previamente. Se agrega una columna nueva al almacén analítico, `address.object.streetNo.string`, donde se almacena este valor "123".

**Asignación de un tipo de datos a un sufijo**

Esta es la asignación de los tipos de datos de propiedad y sus representaciones de sufijos en el almacén analítico:

|Tipo de datos original  |Sufijo  |Ejemplo  |
|---------|---------|---------|
| Doble |  ".float64" |    24.99|
| Array | ".array" |    ["a", "b"]|
|Binary | ".binary" |0|
|Boolean    | ".bool"   |Verdadero|
|Int32  | ".int32"  |123|
|Int64  | ".int64"  |255486129307|
|Null   | ".null"   | null|
|String|    ".string" | "ABC"|
|Timestamp |    ".timestamp" |  Timestamp(0, 0)|
|DateTime   |".date"    | ISODate("2020-08-21T07:43:07.375Z")|
|ObjectId   |".objectId"    | ObjectId("5f3f7b59330ec25c132623a2")|
|Documento   |".object" |    {"a": "a"}|

* Se espera un comportamiento diferente con respecto a los valores explícitos `null`:
  * Los grupos de Spark en Azure Synapse leerán estos valores como `0` (cero).
  * Los grupos sin servidor SQL de Azure Synapse leerán estos valores como `NULL`.
  
* Se espera un comportamiento diferente con respecto a las columnas que faltan:
  * Los grupos de Spark en Azure Synapse representarán estas columnas como `undefined`.
  * Los grupos sin servidor de SQL en Azure Synapse representarán estas columnas como `NULL`.

## <a name="cost-effective-archival-of-historical-data"></a>Archivado rentable de datos históricos

La organización en capas de datos hace referencia a la separación de los datos entre las infraestructuras de almacenamiento optimizadas para los distintos escenarios. Con lo que se mejora el rendimiento general y la rentabilidad de la pila de datos de un extremo a otro. Con el almacén analítico, Azure Cosmos DB ahora admite la organización automática en capas de datos desde el almacén transaccional hacia el almacén analítico con diferentes diseños de datos. Con el almacén analítico optimizado en cuanto al costo de almacenamiento en comparación con el almacén transaccional, usted puede conservar un horizonte mucho más largo de datos operativos para el análisis histórico.

Una vez que el almacén analítico esté habilitado, en función de las necesidades de retención de datos de las cargas de trabajo transaccionales, puede configurar la propiedad "Período de vida del almacén transaccional (TTL transaccional)" para que los registros se eliminen automáticamente del almacén transaccional después de un período de tiempo determinado. Del mismo modo, el "Período de vida del almacén analítico (TTL analítico)" le permite administrar el ciclo de vida de los datos retenidos en el almacén analítico de manera independiente del almacén transaccional. Al habilitar el almacén analítico y configurar las propiedades TTL, puede establecer un nivel y definir sin problemas el período de retención de los datos para los dos almacenes.

> [!NOTE]
>Actualmente, el almacén analítico no admite copia de seguridad y restauración. No se puede planear la directiva de copia de seguridad basándose en el almacén analítico. Para más información, consulte la sección de limitaciones de [este](synapse-link.md#limitations) documento. Es importante tener en cuenta que los datos del almacén analítico tienen un esquema diferente al que existe en el almacén transaccional. Aunque puede generar instantáneas de los datos del almacén analítico, sin costos de RU, no se puede garantizar el uso de esta instantánea para volver a generar el almacén transaccional. Este proceso no se admite.

## <a name="global-distribution"></a>Distribución global

Si tiene una cuenta de Azure Cosmos DB distribuida globalmente, después de habilitar el almacén analítico para un contenedor, estará disponible en todas las regiones de dicha cuenta.  Los cambios en los datos operativos se replican globalmente en todas las regiones. Puede ejecutar consultas analíticas de forma eficaz en la copia regional más cercana de los datos en Azure Cosmos DB.

## <a name="security"></a>Seguridad

La autenticación con el almacén analítico es igual que en un almacén transaccional para una base de datos determinada. Puede usar claves principales o de solo lectura para la autenticación. Puede aprovechar el servicio vinculado en Synapse Studio para evitar pegar las claves de Azure Cosmos DB en los cuadernos de Spark. El acceso a este servicio vinculado está disponible para todos los usuarios que tengan acceso al área de trabajo.

## <a name="support-for-multiple-azure-synapse-analytics-runtimes"></a>Compatibilidad con varios runtimes de Azure Synapse Analytics

El almacén analítico está optimizado para proporcionar escalabilidad, elasticidad y rendimiento para las cargas de trabajo analíticas sin depender de los runtimes de proceso. La tecnología de almacenamiento se administra automáticamente para optimizar las cargas de trabajo analíticas sin esfuerzo manual.

Al desacoplar el sistema de almacenamiento analítico del sistema de proceso analítico, los datos en el almacén analítico de Azure Cosmos DB se pueden consultar simultáneamente desde los distintos runtimes analíticos admitidos por Azure Synapse Analytics. En la actualidad, Azure Synapse Analytics admite Apache Spark y el grupo de SQL sin servidor con el almacén analítico de Azure Cosmos DB.

> [!NOTE]
> Solo se puede leer del almacén analítico mediante los entornos de ejecución de Azure Synapse Analytics. Y al contrario, los entornos de ejecución de Azure Synapse Analytics solo pueden leer del almacén analítico. Solo el proceso de sincronización automática puede cambiar los datos en el almacén analítico. Puede volver a escribir datos en el almacén transaccional de Cosmos DB mediante el grupo de Azure Synapse Analytics, por medio del SDK de OLPT integrado de Azure Cosmos DB.

## <a name="pricing"></a><a id="analytical-store-pricing"></a> Precios

El almacén analítico sigue un modelo de precios basado en el consumo, donde se le cobra del modo siguiente:

* Almacenamiento: El volumen de los datos retenidos en el almacén analítico cada mes, incluidos los datos históricos, tal como se define en el TTL analítico.

* Operaciones de escritura analíticas: Sincronización totalmente administrada de las actualizaciones de datos operativos en el almacén analítico desde el almacén transaccional (sincronización automática).

* Operaciones de lectura analíticas: operaciones de lectura realizadas en el almacén analítico desde los tiempos de ejecución del grupo de SQL sin servidor y el grupo de Spark de Azure Synapse Analytics.

Los precios del almacén analítico son independientes del modelo de precios del almacén transaccional. No hay ningún concepto de RU aprovisionadas en el almacén analítico. Consulte la [página de precios de Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/) para obtener información completa sobre el modelo de precios del almacén analítico.

Para obtener una estimación general del costo de habilitación del almacén analítico en un contenedor de Azure Cosmos DB, puede usar el [planificador de capacidad de Azure Cosmos DB](https://cosmos.azure.com/capacitycalculator/) y obtener una estimación de los costos de almacenamiento analítico y de las operaciones de escritura. Los costos de las operaciones de lectura analíticas dependen de las características de la carga de trabajo analítica, pero como estimación general, el análisis de 1 TB de datos en el almacén analítico suele generar 130 000 operaciones de lectura analíticas, con un costo de 0,065 USD.

> [!NOTE]
> Las estimaciones de operaciones de lectura del almacén analítico no se incluyen en la calculadora de costos de Cosmos DB, ya que son una función de la carga de trabajo analítica. Aunque la estimación anterior es para examinar 1 TB de datos en el almacén analítico, la aplicación de filtros reduce el volumen de datos analizados y esto determina el número exacto de operaciones de lectura analíticas según el modelo de precios de consumo. Una prueba de concepto en torno a la carga de trabajo analítica proporcionaría una estimación más fina de las operaciones de lectura analítica.


## <a name="analytical-time-to-live-ttl"></a><a id="analytical-ttl"></a> Período de vida (TTL) analítico

El TTL analítico indica cuánto tiempo deben conservarse los datos en el almacén analítico para un contenedor dado. 

Si el almacén analítico está habilitado, las inserciones, actualizaciones y eliminaciones de datos operativos se sincronizan automáticamente desde el almacén transaccional hacia el almacén analítico, independientemente de la configuración de TTL transaccional. La retención de estos datos operativos en el almacén analítico puede controlarse mediante el valor de TTL analítico en el nivel de contenedor, tal como se especifica a continuación:

El TTL analítico en un contenedor se establece mediante la propiedad `AnalyticalStoreTimeToLiveInSeconds`:

* Si el valor se establece en "0", falta (o se establece en NULL): el almacén analítico se deshabilita y no se replica ningún dato del almacén transaccional al almacén analítico.

* Si está presente y el valor se establece en "-1": el almacén analítico conserva todos los datos históricos, independientemente de la retención de los datos en el almacén transaccional. Este valor indica que el almacén analítico tiene una retención infinita de los datos operativos.

* Si está presente y el valor se establece en algún número positivo "n": los elementos expirarán del almacén analítico "n" segundos después de la hora de la última modificación en el almacén transaccional. Este valor se puede aprovechar si desea conservar los datos operativos durante un período de tiempo limitado en el almacén analítico, independientemente de la retención de los datos en el almacén transaccional.

Algunos puntos que se deben tener en cuenta:

*   Después de habilitar el almacén analítico con un valor de TTL analítico, puede actualizarse a un valor válido diferente más tarde. 
*   Aunque el TTL transaccional se puede establecer a nivel de contenedor o de elemento, actualmente el TTL analítico solo se puede establecer a nivel de contenedor.
*   Puede lograr una retención más prolongada de los datos operativos en el almacén analítico si establece un TTL analítico >= TTL transaccional a nivel de contenedor.
*   Se puede hacer que el almacén analítico refleje el almacén transaccional si se establece lo siguiente: TTL analítico = TTL transaccional.

Cómo habilitar el almacén analítico en un contenedor:

* En Azure Portal, la opción de TTL analítico, cuando está activada, se establece en el valor predeterminado de -1. Puede cambiar este valor a "n" segundos; para ello, vaya a la configuración del contenedor en el Explorador de datos. 
 
* Desde el SDK de Azure Management, los SDK de Azure Cosmos DB, PowerShell o la CLI, la opción de TTL analítico se puede habilitar estableciendo su valor en -1 o "n" segundos. 

Para obtener más información, consulte [Configuración del TTL analítico en un contenedor](configure-synapse-link.md#create-analytical-ttl).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte la siguiente documentación:

* [Azure Synapse Link para Azure Cosmos DB](synapse-link.md)

* [Introducción a Azure Synapse Link para Azure Cosmos DB](configure-synapse-link.md)

* [Preguntas frecuentes sobre Synapse Link para Azure Cosmos DB](synapse-link-frequently-asked-questions.yml)

* [Casos de uso de Azure Synapse Link para Azure Cosmos DB](synapse-link-use-cases.md)
