---
title: Asignación de esquemas y tipos de datos en la actividad de copia
description: Obtenga información acerca de cómo la actividad de copia de Azure Data Factory asigna esquemas y tipos de datos desde datos de origen hasta datos receptores.
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.date: 06/22/2020
ms.author: jianleishen
ms.openlocfilehash: 2bd616ddec207d2aad47608c6f0200c7b629471e
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109482532"
---
# <a name="schema-and-data-type-mapping-in-copy-activity"></a>Asignación de esquemas y tipos de datos en la actividad de copia
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se describe el modo en que la actividad de copia de Azure Data Factory realiza la asignación de esquemas y la asignación del tipo de datos desde datos de origen hasta datos receptores.

## <a name="schema-mapping"></a>Asignación de esquemas

### <a name="default-mapping"></a>Asignación predeterminada

De forma predeterminada, la actividad de copia asigna los datos de origen al receptor **por nombres de columna** con distinción de mayúsculas y minúsculas. Si el receptor no existe, por ejemplo, al escribir en los archivos, los nombres de los campos de origen se conservarán como nombres de receptor. Si el receptor ya existe, debe contener todas las columnas que se copien del origen. Esta asignación predeterminada admite esquemas flexibles y derivación de esquemas del origen al receptor de una ejecución a otra: todos los datos devueltos por el almacén de datos de origen se pueden copiar en el receptor.

Si el origen es un archivo de texto sin línea de encabezado, se requiere una [asignación explícita](#explicit-mapping), ya que el origen no contiene nombres de columna.

### <a name="explicit-mapping"></a>Asignación explícita

También puede especificar una asignación explícita para personalizar la asignación de columnas o campos del origen al receptor según sus necesidades. Con la asignación explícita, solo puede copiar parcialmente los datos de origen en el receptor o asignar los datos de origen al receptor con nombres diferentes, o incluso volver a conformar los datos tabulares y jerárquicos. Actividad de copia:

1. Lee los datos desde el origen y determina el esquema de origen.
2. Aplica la asignación definida.
3. Escribe los datos en el receptor.

Más información sobre:

- [Origen tabular a un receptor tabular](#tabular-source-to-tabular-sink)
- [Origen jerárquico a un receptor tabular](#hierarchical-source-to-tabular-sink)
- [Origen tabular o jerárquico a un receptor jerárquico](#tabularhierarchical-source-to-hierarchical-sink)

Puede configurar la asignación en la interfaz de usuario de creación de Data Factory > actividad de copia > pestaña Asignación, o bien especificar mediante programación la asignación en la propiedad `translator` de la actividad de copia. Las siguientes propiedades se admiten en `translator` -> matriz `mappings` -> objetos -> `source` y `sink`, que apuntan a la columna o campo específico para asignar datos.

| Propiedad | Descripción                                                  | Obligatorio |
| -------- | ------------------------------------------------------------ | -------- |
| name     | Nombre de la columna o campo de origen o receptor. Aplica para el origen y el receptor tabulares. | Sí      |
| ordinal  | Índice de columna. Inicia desde 1. <br>Se aplica y es obligatoria cuando se usa texto delimitado sin línea de encabezado. | No       |
| path     | Expresión de ruta de acceso JSON de cada campo para su extracción o asignación. Aplica para el origen y el receptor jerárquicos; por ejemplo, Cosmos DB, MongoDB o conectores REST.<br>Para los campos situados bajo el objeto raíz, la ruta de acceso JSON comienza con la raíz `$`; para los campos incluidos dentro de la matriz elegida mediante la propiedad `collectionReference`, la ruta de acceso JSON empieza desde el elemento de matriz sin `$`. | No       |
| type     | Tipo de datos provisionales de Data Factory de la columna de origen o receptor. En general, no es necesario especificar ni cambiar esta propiedad. Más información sobre la [asignación de tipo de datos](#data-type-mapping). | No       |
| culture  | Cultura de la columna de origen o receptor. Se aplica cuando el tipo es `Datetime` o `Datetimeoffset`. El valor predeterminado es `en-us`.<br>En general, no es necesario especificar ni cambiar esta propiedad. Más información sobre la [asignación de tipo de datos](#data-type-mapping). | No       |
| format   | Cadena de formato que se usa cuando el tipo es `Datetime` o `Datetimeoffset`. Consulte [Cadenas con formato de fecha y hora personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings) para obtener información sobre el formato de fecha y hora. En general, no es necesario especificar ni cambiar esta propiedad. Más información sobre la [asignación de tipo de datos](#data-type-mapping). | No       |

Se admiten las siguientes propiedades en `translator` además de `mappings`:

| Propiedad            | Descripción                                                  | Obligatorio |
| ------------------- | ------------------------------------------------------------ | -------- |
| collectionReference | Aplica al copiar datos de un origen jerárquico; por ejemplo, Cosmos DB, MongoDB o conectores REST.<br>Si desea iterar y extraer datos de los objetos **dentro de un campo de matriz** con el mismo patrón y convertir al modo por fila por objeto, especifique la ruta de acceso JSON de esa matriz para realizar la aplicación cruzada. | No       |

#### <a name="tabular-source-to-tabular-sink"></a>Origen tabular a un receptor tabular

Por ejemplo, para copiar datos de Salesforce a Azure SQL Database y asignar explícitamente tres columnas:

1. En la pestaña actividad de copia > asignación, haga clic en el botón **Importar esquemas** para importar los esquemas de origen y de receptor.

2. Asigne los campos necesarios y excluya o elimine los demás.

![Asignación de tabular a tabular](media/copy-activity-schema-and-type-mapping/map-tabular-to-tabular.png)

La misma asignación se puede configurar de la siguiente forma en la carga de la actividad de copia (consulte `translator`):

```json
{
    "name": "CopyActivityTabularToTabular",
    "type": "Copy",
    "typeProperties": {
        "source": { "type": "SalesforceSource" },
        "sink": { "type": "SqlSink" },
        "translator": {
            "type": "TabularTranslator",
            "mappings": [
                {
                    "source": { "name": "Id" },
                    "sink": { "name": "CustomerID" }
                },
                {
                    "source": { "name": "Name" },
                    "sink": { "name": "LastName" }
                },
                {
                    "source": { "name": "LastModifiedDate" },
                    "sink": { "name": "ModifiedDate" }
                }
            ]
        }
    },
    ...
}
```

Para copiar datos de los archivos de texto delimitados sin una línea de encabezado, las columnas se representan mediante el índice, en lugar de los nombres. 

```json
{
    "name": "CopyActivityTabularToTabular",
    "type": "Copy",
    "typeProperties": {
        "source": { "type": "DelimitedTextSource" },
        "sink": { "type": "SqlSink" },
        "translator": {
            "type": "TabularTranslator",
            "mappings": [
                {
                    "source": { "ordinal": "1" },
                    "sink": { "name": "CustomerID" }
                }, 
                {
                    "source": { "ordinal": "2" },
                    "sink": { "name": "LastName" }
                }, 
                {
                    "source": { "ordinal": "3" },
                    "sink": { "name": "ModifiedDate" }
                }
            ]
        }
    },
    ...
}
```

#### <a name="hierarchical-source-to-tabular-sink"></a>Origen jerárquico a un receptor tabular

Al copiar datos de un origen jerárquico a un receptor tabular, la actividad de copia admite las siguientes funcionalidades:

- Extraer datos de objetos y matrices.
- Usar CROSS APPLY en varios objetos con el mismo patrón de una matriz, en cuyo caso se convierte un objeto JSON en varios registros del resultado tabular.

Para una transformación de datos jerárquicos a tabulares más avanzada, puede usar [Data Flow](concepts-data-flow-overview.md). 

Por ejemplo, si tiene un documento de origen de MongoDB con el siguiente contenido:

```json
{
    "id": {
        "$oid": "592e07800000000000000000"
    },
    "number": "01",
    "date": "20170122",
    "orders": [
        {
            "prod": "p1",
            "price": 23
        },
        {
            "prod": "p2",
            "price": 13
        },
        {
            "prod": "p3",
            "price": 231
        }
    ],
    "city": [ { "name": "Seattle" } ]
}
```

Y quiere copiarlo en un archivo de texto del formato siguiente con línea de encabezado, puede hacerlo mediante el acoplamiento de los datos dentro de la matriz *(order_pd y order_price)* y la combinación cruzada con la información raíz en común *(número, fecha y ciudad)* :

| orderNumber | orderDate | order_pd | order_price | city    |
| ----------- | --------- | -------- | ----------- | ------- |
| 01          | 20170122  | P1       | 23          | Seattle |
| 01          | 20170122  | P2       | 13          | Seattle |
| 01          | 20170122  | P3       | 231         | Seattle |

Puede definir dicha asignación en la interfaz de usuario de creación de Data Factory:

1. En la pestaña actividad de copia > asignación, haga clic en el botón **Importar esquemas** para importar los esquemas de origen y de receptor. Ya que Data Factory muestrea los objetos más importantes al importar el esquema, si no aparece algún campo, puede agregarlo a la capa correcta en la jerarquía; para ello, mantenga el puntero sobre un nombre de campo existente y elija la opción para agregar un nodo, un objeto o una matriz.

2. Seleccione la matriz en la que quiere iterar y extraer datos. Se cumplimentará automáticamente como una **referencia de colecciones**. Nota: Solo se admite una matriz única para esta operación.

3. Asigne los campos necesarios al receptor. Data Factory determina automáticamente las rutas de acceso JSON correspondientes para el lado jerárquico.

> [!NOTE]
> El registro se omite para los registros en los que la matriz marcada como referencia de la colección está vacía y la casilla está activada.

![Asignación de datos jerárquicos a tabulares mediante la interfaz de usuario](media/copy-activity-schema-and-type-mapping/map-hierarchical-to-tabular-ui.png)

También puede cambiar al **Editor avanzado**, en cuyo caso puede ver y editar directamente las rutas de acceso JSON de los campos. Si decide agregar una nueva asignación en esta vista, especifique la ruta de acceso JSON.

![Asignación de datos jerárquicos a tabulares mediante el editor avanzado](media/copy-activity-schema-and-type-mapping/map-hierarchical-to-tabular-advanced-editor.png)

La misma asignación se puede configurar de la siguiente forma en la carga de la actividad de copia (consulte `translator`):


```json
{
    "name": "CopyActivityHierarchicalToTabular",
    "type": "Copy",
    "typeProperties": {
        "source": { "type": "MongoDbV2Source" },
        "sink": { "type": "DelimitedTextSink" },
        "translator": {
            "type": "TabularTranslator",
            "mappings": [
                {
                    "source": { "path": "$['number']" },
                    "sink": { "name": "orderNumber" }
                },
                {
                    "source": { "path": "$['date']" },
                    "sink": { "name": "orderDate" }
                },
                {
                    "source": { "path": "['prod']" },
                    "sink": { "name": "order_pd" }
                },
                {
                    "source": { "path": "['price']" },
                    "sink": { "name": "order_price" }
                },
                {
                    "source": { "path": "$['city'][0]['name']" },
                    "sink": { "name": "city" }
                }
            ],
            "collectionReference": "$['orders']"
        }
    },
    ...
}
```

#### <a name="tabularhierarchical-source-to-hierarchical-sink"></a>Origen tabular o jerárquico a un receptor jerárquico

El flujo de la experiencia del usuario es similar a el de [Origen jerárquico a receptor tabular](#hierarchical-source-to-tabular-sink). 

Al copiar datos del origen tabular al receptor jerárquico, no se admite la escritura en una matriz dentro del objeto.

Al copiar datos de un origen jerárquico a un receptor jerárquico, también puede conservar la jerarquía de todo un nivel si selecciona el objeto o la matriz y lo asigna al receptor sin modificar los campos internos.

Para una transformación de conformación de datos más avanzada, puede usar [Data Flow](concepts-data-flow-overview.md). 

### <a name="parameterize-mapping"></a>Parametrización de asignaciones

Si quiere crear una canalización en plantilla para copiar dinámicamente un gran número de objetos, determine si puede aprovechar la [asignación predeterminada](#default-mapping) o si necesita definir una [asignación explícita](#explicit-mapping) para los objetos respectivos.

Si se necesita una asignación explícita, puede:

1. Definir un parámetro con un tipo de objeto en el nivel de canalización; por ejemplo, `mapping`.

2. Parametrizar la asignación: en la actividad de copia > pestaña Asignación, elija agregar contenido dinámico y seleccione el parámetro anterior. La carga de la actividad tendría un aspecto como el siguiente:

    ```json
    {
        "name": "CopyActivityHierarchicalToTabular",
        "type": "Copy",
        "typeProperties": {
            "source": {...},
            "sink": {...},
            "translator": {
                "value": "@pipeline().parameters.mapping",
                "type": "Expression"
            },
            ...
        }
    }
    ```

3. Construir el valor que se va a pasar al parámetro de asignación. Debe ser el objeto completo de la definición de `translator`; consulte los ejemplos de la sección [asignación explícita](#explicit-mapping). Por ejemplo, para copiar de un origen tabular a un receptor tabular, el valor debe ser `{"type":"TabularTranslator","mappings":[{"source":{"name":"Id"},"sink":{"name":"CustomerID"}},{"source":{"name":"Name"},"sink":{"name":"LastName"}},{"source":{"name":"LastModifiedDate"},"sink":{"name":"ModifiedDate"}}]}`.

## <a name="data-type-mapping"></a>Asignación de tipos de datos

La actividad de copia realiza la asignación de tipos de origen a tipos de receptores con el siguiente flujo: 

1. Convierta los tipos de datos nativos del origen a tipos de datos provisionales de Azure Data Factory.
2. Convierta automáticamente el tipo de datos provisional según sea necesario para que coincida con los tipos de receptor correspondientes; se aplica para la [asignación predeterminada](#default-mapping) y la [asignación explícita](#explicit-mapping).
3. Convierta los tipos de datos provisionales de Data Factory en tipos de datos nativos del receptor.

La actividad de copia actualmente admite los siguientes tipos de datos provisionales: booleano, byte, matriz de bytes, DateTime, DatetimeOffset, decimal, double, GUID, Int16, Int32, Int64, SByte, single, cadena, intervalo de tiempo, UInt16, UInt32, and UInt64.

Se admiten las siguientes conversiones de tipos de datos entre los tipos provisionales de origen a receptor.

| Origen/receptor | Boolean | Byte array | Decimal | Fecha u hora <small>(1)</small> | <small>Con punto flotante (2)</small> | GUID | Entero <small>(3)</small> | String | TimeSpan |
| ----------- | ------- | ---------- | ------- | ---------------------------- | ------------------------------ | ---- | -------------------------- | ------ | -------- |
| Boolean     | ✓       |            | ✓       |                              | ✓                              |      | ✓                          | ✓      |          |
| Byte array  |         | ✓          |         |                              |                                |      |                            | ✓      |          |
| Fecha y hora   |         |            |         | ✓                            |                                |      |                            | ✓      |          |
| Decimal     | ✓       |            | ✓       |                              | ✓                              |      | ✓                          | ✓      |          |
| Con punto flotante | ✓       |            | ✓       |                              | ✓                              |      | ✓                          | ✓      |          |
| GUID        |         |            |         |                              |                                | ✓    |                            | ✓      |          |
| Entero     | ✓       |            | ✓       |                              | ✓                              |      | ✓                          | ✓      |          |
| String      | ✓       | ✓          | ✓       | ✓                            | ✓                              | ✓    | ✓                          | ✓      | ✓        |
| TimeSpan    |         |            |         |                              |                                |      |                            | ✓      | ✓        |

(1) Los valores de fecha y hora incluyen a DateTime y DateTimeOffset.

(2) Los valores con punto flotante incluyen a Single y Double.

(3) Los valores enteros incluyen a SByte, Byte, Int16, UInt16, Int32, UInt32, Int64 y UInt64.

> [!NOTE]
> - Actualmente, esta conversión de tipos de datos se admite cuando se copia entre datos tabulares. Los orígenes o receptores jerárquicos no son compatible, por lo que no hay ninguna conversión de tipo de datos definida por el sistema entre los tipos provisionales de origen y receptor.
> - Esta característica funciona con el modelo de conjunto de datos más reciente. Si no ve esta opción desde la interfaz de usuario, intente crear un nuevo conjunto de datos.

Las siguientes propiedades son compatibles con la actividad de copia para la conversión de tipos de datos (en la sección `translator` para la creación mediante programación):

| Propiedad                         | Descripción                                                  | Obligatorio |
| -------------------------------- | ------------------------------------------------------------ | -------- |
| typeConversion                   | Habilita la nueva experiencia de conversión de tipos de datos. <br>El valor predeterminado es false debido a la compatibilidad con versiones anteriores.<br><br>En el caso de las nuevas actividades de copia creadas a través de la interfaz de usuario de creación de Data Factory a partir de finales de junio de 2020, esta conversión de tipos de datos está habilitada de forma predeterminada para obtener la mejor experiencia, y puede ver la siguiente configuración de conversión de tipos en actividad de copia > pestaña Asignación para los escenarios aplicables. <br>Para crear una canalización mediante programación, debe establecer explícitamente la propiedad `typeConversion` en true para habilitarla.<br>En el caso de las actividades de copia existentes creadas antes de la publicación de esta característica, no verá las opciones de conversión de tipos en la interfaz de usuario de creación de Data Factory debido a la compatibilidad con versiones anteriores. | No       |
| typeConversionSettings           | Grupo de configuraciones para conversión de tipos. Se aplica cuando `typeConversion` está establecido en `true`. Todas las propiedades siguientes se encuentran en este grupo. | No       |
| *En `typeConversionSettings`* |                                                              |          |
| allowDataTruncation              | Permite truncar los datos al convertir los datos del origen al receptor con un tipo diferente durante la copia; por ejemplo, de decimal a entero, de DatetimeOffset a DateTime. <br>El valor predeterminado es true. | No       |
| treatBooleanAsNumber             | Trata los valores booleanos como números; por ejemplo, true como 1.<br>El valor predeterminado es False. | No       |
| dateTimeFormat                   | Da formato a la cadena al realizar conversiones entre fechas sin desplazamiento de zona horaria y cadenas; por ejemplo, `yyyy-MM-dd HH:mm:ss.fff`.  Consulte [Cadenas con formato de fecha y hora personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings) para obtener información detallada. | No       |
| dateTimeOffsetFormat             | Da formato a la cadena al realizar conversiones entre fechas con desplazamiento de zona horaria y cadenas; por ejemplo, `yyyy-MM-dd HH:mm:ss.fff zzz`.  Consulte [Cadenas con formato de fecha y hora personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings) para obtener información detallada. | No       |
| timeSpanFormat                   | Da formato a la cadena al realizar conversiones entre intervalos de tiempo y cadenas; por ejemplo, `dd\.hh\:mm`. Consulte [Cadenas con formato de intervalo de tiempo personalizado](/dotnet/standard/base-types/custom-timespan-format-strings) para obtener información detallada. | No       |
| culture                          | Información de referencia cultural que se va a usar al convertir tipos; por ejemplo, `en-us` o `fr-fr`. | No       |

**Ejemplo**:

```json
{
    "name": "CopyActivity",
    "type": "Copy",
    "typeProperties": {
        "source": {
            "type": "ParquetSource"
        },
        "sink": {
            "type": "SqlSink"
        },
        "translator": {
            "type": "TabularTranslator",
            "typeConversion": true,
            "typeConversionSettings": {
                "allowDataTruncation": true,
                "treatBooleanAsNumber": true,
                "dateTimeFormat": "yyyy-MM-dd HH:mm:ss.fff",
                "dateTimeOffsetFormat": "yyyy-MM-dd HH:mm:ss.fff zzz",
                "timeSpanFormat": "dd\.hh\:mm",
                "culture": "en-gb"
            }
        }
    },
    ...
}
```

## <a name="legacy-models"></a>Modelos heredados

> [!NOTE]
> Estos modelos para asignar columnas o campos del origen al receptor siguen siendo compatibles con versiones anteriores. Se recomienda usar el nuevo modelo mencionado en [asignación de esquemas](#schema-mapping). La interfaz de usuario de creación en Data Factory se ha cambiado para generar el nuevo modelo.

### <a name="alternative-column-mapping-legacy-model"></a>Asignación de columnas alternativa (modelo heredado)

Puede especificar actividad de copia -> `translator` -> `columnMappings` para realizar asignaciones de datos en forma tabular. En este caso, la sección "structure" es necesaria tanto para los conjuntos de datos de entrada como para los de salida. La asignación de columnas admite **la asignación de todas las columnas o un subconjunto de ellas del conjunto de datos de origen "structure" a todas las columnas del conjunto de datos receptor "structure"** . Las siguientes son las condiciones de error que tienen como resultado una excepción:

- El resultado de la consulta del almacén de datos de origen no tiene un nombre de columna que se especifique en la sección "structure" del conjunto de datos de entrada.
- El almacén de datos receptor (si está en el esquema predefinido) no tiene un nombre de columna que se especifique en la sección "structure" del conjunto de datos de salida.
- O bien menos columnas o más columnas en "structure" del conjunto de datos receptor de las que se especifican en la asignación.
- Asignación duplicada.

En el siguiente ejemplo, el conjunto de datos de entrada tiene una estructura y apunta a una tabla en una base de datos de Oracle local.

```json
{
    "name": "OracleDataset",
    "properties": {
        "structure":
         [
            { "name": "UserId"},
            { "name": "Name"},
            { "name": "Group"}
         ],
        "type": "OracleTable",
        "linkedServiceName": {
            "referenceName": "OracleLinkedService",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "tableName": "SourceTable"
        }
    }
}
```

En este ejemplo, el conjunto de datos de salida tiene una estructura y apunta a una tabla en Salesfoce.

```json
{
    "name": "SalesforceDataset",
    "properties": {
        "structure":
        [
            { "name": "MyUserId"},
            { "name": "MyName" },
            { "name": "MyGroup"}
        ],
        "type": "SalesforceObject",
        "linkedServiceName": {
            "referenceName": "SalesforceLinkedService",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "tableName": "SinkTable"
        }
    }
}
```

El siguiente fragmento JSON define una actividad de copia en una canalización. Las columnas del origen se asignan a columnas del receptor mediante la propiedad **translator** -> **columnMappings**.

```json
{
    "name": "CopyActivity",
    "type": "Copy",
    "inputs": [
        {
            "referenceName": "OracleDataset",
            "type": "DatasetReference"
        }
    ],
    "outputs": [
        {
            "referenceName": "SalesforceDataset",
            "type": "DatasetReference"
        }
    ],
    "typeProperties":    {
        "source": { "type": "OracleSource" },
        "sink": { "type": "SalesforceSink" },
        "translator":
        {
            "type": "TabularTranslator",
            "columnMappings":
            {
                "UserId": "MyUserId",
                "Group": "MyGroup",
                "Name": "MyName"
            }
        }
    }
}
```

Si estaba usando la sintaxis de `"columnMappings": "UserId: MyUserId, Group: MyGroup, Name: MyName"` para especificar una asignación de columnas, esta se sigue admitiendo tal cual.

### <a name="alternative-schema-mapping-legacy-model"></a>Asignación de esquemas alternativa (modelo heredado)

Puede especificar actividad de copia -> `translator` -> `schemaMapping` para realizar la asignación de tablas de datos jerárquicos y datos tabulares. Por ejemplo, al copiar de MongoDB o REST a un archivo de texto, y al copiar de Oracle a la API de Azure Cosmos DB para MongoDB. Se admiten las siguientes propiedades en la sección `translator` de la actividad de copia:

| Propiedad            | Descripción                                                  | Obligatorio |
| :------------------ | :----------------------------------------------------------- | :------- |
| type                | La propiedad type del traductor de la actividad de copia debe establecerse en: **TabularTranslator** | Sí      |
| schemaMapping       | Colección de pares clave-valor, que representa la relación de la asignación **del lado origen al lado receptor**.<br/>- **Clave:** representa el origen. Para un **origen tabular**, especifique el nombre de columna tal como se define en la estructura del conjunto de datos; para un **origen jerárquico**, especifique la expresión de ruta de acceso JSON para todos los campos que va a extraer y asignar.<br>- **Valor:** representa el receptor. Para un **receptor tabular**, especifique el nombre de columna tal como se define en la estructura del conjunto de datos; para un **receptor jerárquico**, especifique la expresión de ruta de acceso JSON para todos los campos que va a extraer y asignar. <br>En el caso de los datos jerárquicos, para los campos en el objeto raíz, la ruta de acceso JSON comienza con root $; para los campos dentro de la matriz elegida mediante la propiedad `collectionReference`, la ruta de acceso JSON empieza desde el elemento de matriz. | Sí      |
| collectionReference | Si desea iterar y extraer datos de los objetos **dentro de un campo de matriz** con el mismo patrón y convertir al modo por fila por objeto, especifique la ruta de acceso JSON de esa matriz para realizar la aplicación cruzada. Esta propiedad solo se admite si el origen son datos jerárquicos. | No       |

**Ejemplo: copia de MongoDB a Oracle:**

Por ejemplo, si tiene un documento de MongoDB con el siguiente contenido:

```json
{
    "id": {
        "$oid": "592e07800000000000000000"
    },
    "number": "01",
    "date": "20170122",
    "orders": [
        {
            "prod": "p1",
            "price": 23
        },
        {
            "prod": "p2",
            "price": 13
        },
        {
            "prod": "p3",
            "price": 231
        }
    ],
    "city": [ { "name": "Seattle" } ]
}
```

y desea copiarlo en una tabla de Azure SQL del formato siguiente, puede hacerlo mediante el acoplamiento de los datos dentro de la matriz *(order_pd y order_price)* y la combinación cruzada con la información de la raíz habitual *(número, fecha y ciudad)* :

| orderNumber | orderDate | order_pd | order_price | city    |
| ----------- | --------- | -------- | ----------- | ------- |
| 01          | 20170122  | P1       | 23          | Seattle |
| 01          | 20170122  | P2       | 13          | Seattle |
| 01          | 20170122  | P3       | 231         | Seattle |

Configure la regla de asignación de esquemas como el siguiente ejemplo JSON de actividad de copia:

```json
{
    "name": "CopyFromMongoDBToOracle",
    "type": "Copy",
    "typeProperties": {
        "source": {
            "type": "MongoDbV2Source"
        },
        "sink": {
            "type": "OracleSink"
        },
        "translator": {
            "type": "TabularTranslator",
            "schemaMapping": {
                "$.number": "orderNumber",
                "$.date": "orderDate",
                "prod": "order_pd",
                "price": "order_price",
                "$.city[0].name": "city"
            },
            "collectionReference":  "$.orders"
        }
    }
}
```

## <a name="next-steps"></a>Pasos siguientes
Consulte los otros artículos de la actividad de copia:

- [Información general de la actividad de copia](copy-activity-overview.md)
