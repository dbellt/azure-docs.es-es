---
title: Formatos de archivo admitidos en Azure Data Factory (heredados)
description: En este tema se describen los formatos de archivo y los códigos de compresión que admiten los conectores basados en archivos en Azure Data Factory.
author: jianleishen
ms.author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.date: 12/10/2019
ms.openlocfilehash: 960c2a995a6e68b2f15e553121bc860f9ac29890
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109488526"
---
# <a name="supported-file-formats-and-compression-codecs-in-azure-data-factory-legacy"></a>Formatos de archivo y códecs de compresión admitidos en Azure Data Factory (heredados)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

*Este artículo se aplica a los conectores siguientes: [Amazon S3](connector-amazon-simple-storage-service.md), [Azure Blob](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md), [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md), [Azure File Storage](connector-azure-file-storage.md), [File System](connector-file-system.md), [FTP](connector-ftp.md), [Google Cloud Storage](connector-google-cloud-storage.md), [HDFS](connector-hdfs.md), [HTTP](connector-http.md) y [SFTP](connector-sftp.md).*

>[!IMPORTANT]
>Data Factory ha presentado el nuevo modelo de conjuntos de datos basado en formato. Consulte el artículo sobre formato correspondiente con los detalles. <br>- [Formato Avro](format-avro.md)<br>- [Formato binario](format-binary.md)<br>- [Formato de texto delimitado](format-delimited-text.md)<br>- [Formato JSON](format-json.md)<br>- [Formato ORC](format-orc.md)<br>- [Formato Parquet](format-parquet.md)<br>Todavía se admiten el resto de configuraciones mencionadas en este artículo tal y como están por motivos de compatibilidad con versiones anteriores. Se sugiere usar el nuevo modelo de aquí en adelante. 

## <a name="text-format-legacy"></a><a name="text-format"></a> Formato de texto (heredado)

>[!NOTE]
>Obtenga información sobre el nuevo modelo en el artículo [Formato de texto delimitado](format-delimited-text.md). Todavía se admiten las siguientes configuraciones en el conjunto de datos basado en archivo, porque cuentan con compatibilidad con versiones anteriores. Se sugiere usar el nuevo modelo de aquí en adelante.

Si quiere leer un archivo de texto o escribir en él, establezca la propiedad `type` de la sección `format` del conjunto de datos en **TextFormat**. También puede especificar las siguientes propiedades **opcionales** en la sección `format`. Consulte la sección [Ejemplo de TextFormat](#textformat-example) sobre cómo realizar la configuración.

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| columnDelimiter |El carácter utilizado para separar las columnas en un archivo. Puede considerar el uso de un carácter imprimible poco frecuente que es probable que no exista en sus datos. Por ejemplo, especifique "\u0001", que representa el inicio de encabezado (SOH). |Solo se permite un carácter. El valor **predeterminado** es **coma (",")** . <br/><br/>Para usar un carácter Unicode, consulte la [lista de caracteres Unicode](https://en.wikipedia.org/wiki/List_of_Unicode_characters) para obtener el código correspondiente. |No |
| rowDelimiter |carácter utilizado para separar filas en un archivo. |Solo se permite un carácter. El valor **predeterminado** es cualquiera de los siguientes en lectura: **["\r\n", "\r", "\n"]** y **"\r\n"** en escritura. |No |
| escapeChar |carácter especial utilizado para aplicar una secuencia de escape a un delimitador de columna en el contenido de un archivo de entrada. <br/><br/>No puede especificar ambos valores escapeChar y quoteChar para una tabla. |Solo se permite un carácter. Ningún valor predeterminado. <br/><br/>Ejemplo: si usa la coma (",") como delimitador de columna, pero quiere tener el carácter de coma en el texto (ejemplo: "Hello, world"), puede definir "$" como carácter de escape y usar la cadena "Hello$, world" en el origen. |No |
| quoteChar |carácter utilizado para citar el valor de una cadena. Los delimitadores de columna y fila de dentro de las comillas se tratan como parte del valor de la cadena. Esta propiedad es aplicable tanto al conjunto de datos de entrada como al de salida.<br/><br/>No puede especificar ambos valores escapeChar y quoteChar para una tabla. |Solo se permite un carácter. Ningún valor predeterminado. <br/><br/>Por ejemplo, si tiene la coma (',') como delimitador de columna, pero quiere tener el carácter de coma en el texto (por ejemplo: <Hello, world>), puede definir " (comillas dobles) como comillas y usar la cadena "Hello, world" en el origen. |No |
| nullValue |uno o varios caracteres empleados para representar un valor nulo. |Uno o más caracteres. Los valores **predeterminados** son **"\N" y "NULL"** en lectura y **"\N"** en escritura. |No |
| encodingName |permite especificar el nombre de la codificación. |Un nombre de codificación válido. Consulte la [propiedad Encoding.EncodingName](/dotnet/api/system.text.encoding). Por ejemplo: windows-1250 o shift_jis. El valor **predeterminado** es **UTF-8**. |No |
| firstRowAsHeader |Especifica si se tendrá en cuenta la primera fila como encabezado. Para un conjunto de datos de entrada, Data Factory lee la primera fila como encabezado. Para un conjunto de datos de salida, Data Factory escribe la primera fila como encabezado. <br/><br/>Consulte [Escenarios de uso `firstRowAsHeader` y `skipLineCount`](#scenarios-for-using-firstrowasheader-and-skiplinecount) para ver ejemplos de escenarios. |True<br/><b>False (valor predeterminado)</b> |No |
| skipLineCount |Indica el número de filas **no vacías** que se omitirán al leer datos de archivos de entrada. Si se especifican ambos valores skipLineCount y firstRowAsHeader, las líneas se omiten primero y, luego, la información del encabezado se lee a partir del archivo de entrada. <br/><br/>Consulte [Escenarios de uso `firstRowAsHeader` y `skipLineCount`](#scenarios-for-using-firstrowasheader-and-skiplinecount) para ver ejemplos de escenarios. |Entero |No |
| treatEmptyAsNull |permite especificar si hay que tratar una cadena nula o vacía como un valor nulo al leer datos de un archivo de entrada. |**True (predeterminado)**<br/>False |No |

### <a name="textformat-example"></a>Ejemplo de TextFormat

En la siguiente definición JSON de un conjunto de datos, se especifican algunas propiedades opcionales.

```json
"typeProperties":
{
    "folderPath": "mycontainer/myfolder",
    "fileName": "myblobname",
    "format":
    {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": ";",
        "quoteChar": "\"",
        "NullValue": "NaN",
        "firstRowAsHeader": true,
        "skipLineCount": 0,
        "treatEmptyAsNull": true
    }
},
```

Para usar un `escapeChar` en lugar de `quoteChar`, reemplace la línea con `quoteChar` por el siguiente escapeChar:

```json
"escapeChar": "$",
```

### <a name="scenarios-for-using-firstrowasheader-and-skiplinecount"></a>Escenarios de uso de firstRowAsHeader y skipLineCount

* Va a copiar de un origen que no es archivo a un archivo de texto y quiere agregar una línea de encabezado que contenga los metadatos de esquema (por ejemplo, esquema SQL). Especifique `firstRowAsHeader` como true en el conjunto de datos de salida de este escenario.
* Va a copiar de un archivo de texto que contiene una línea de encabezado a un receptor que no es archivo y quiere eliminar esa línea. Especifique `firstRowAsHeader` como true en el conjunto de datos de entrada.
* Va a copiar de un archivo de texto y quiere omitir unas cuantas líneas al comienzo que no contienen datos ni información de encabezado. Especifique `skipLineCount` para indicar el número de líneas que se omitirá. Si el resto del archivo contiene una línea de encabezado, también puede especificar `firstRowAsHeader`. Si se especifican `skipLineCount` y `firstRowAsHeader`, las líneas se omiten primero y luego la información del encabezado se lee del archivo de entrada.

## <a name="json-format-legacy"></a><a name="json-format"></a> Formato JSON (heredado)

>[!NOTE]
>Obtenga información sobre el nuevo modelo en el artículo [Formato JSON](format-json.md). Todavía se admiten las siguientes configuraciones en el conjunto de datos basado en archivo, porque cuentan con compatibilidad con versiones anteriores. Se sugiere usar el nuevo modelo de aquí en adelante.

Para **importar o exportar un archivo JSON como está en Azure Cosmos DB**, consulte la sección sobre la importación o exportación de documentos JSON en el artículo [Move data to/from Azure Cosmos DB](connector-azure-cosmos-db.md) (Movimiento de datos a y desde Azure Cosmos DB).

Si quiere analizar los archivos JSON o escribir los datos en formato JSON, establezca la propiedad `type``format` en **JsonFormat**. También puede especificar las siguientes propiedades **opcionales** en la sección `format`. Consulte la sección [Ejemplo de JsonFormat](#jsonformat-example) sobre cómo realizar la configuración.

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| filePattern |Indica el patrón de los datos almacenados en cada archivo JSON. Estos son los valores permitidos: **setOfObjects** y **arrayOfObjects**. El valor **predeterminado** es **setOfObjects**. Consulte la sección [patrones de archivo JSON](#json-file-patterns) para obtener más información acerca de estos patrones. |No |
| jsonNodeReference | Si desea iterar y extraer datos de los objetos dentro de un campo de matriz con el mismo patrón, especifique la ruta de acceso JSON de esa matriz. Esta propiedad se admite solo cuando se copian datos **desde** los archivos JSON. | No |
| jsonPathDefinition | Especifique la expresión de ruta de acceso JSON para cada asignación de columna con un nombre de columna personalizado (que empiece con minúscula). Esta propiedad se admite solo cuando se copian datos **desde** archivos JSON y puede extraer datos del objeto o matriz. <br/><br/> Para los campos en el objeto raíz, comience por root $; para los campos dentro de la matriz elegida mediante la propiedad `jsonNodeReference`, empiece desde el elemento de matriz. Consulte la sección [Ejemplo de JsonFormat](#jsonformat-example) sobre cómo realizar la configuración. | No |
| encodingName |permite especificar el nombre de la codificación. Para obtener la lista de nombres de codificación válidos, consulte la propiedad [Encoding.EncodingName](/dotnet/api/system.text.encoding). Por ejemplo: windows-1250 o shift_jis. El valor **predeterminado** es: **UTF-8**. |No |
| nestingSeparator |Carácter que se usa para separar los niveles de anidamiento. El valor predeterminado es '.' (punto). |No |

>[!NOTE]
>En caso de la aplicación cruzada de los datos en la matriz en varias filas (caso 1 -> ejemplo 2 en los [ejemplos de JsonFormat](#jsonformat-example)), solo puede elegir expandir la matriz con la propiedad `jsonNodeReference`.

### <a name="json-file-patterns"></a>Patrones de archivo JSON

La actividad de copia puede analizar los siguientes patrones de archivos JSON:

- **Tipo I: setOfObjects**

    Cada archivo contiene un único objeto o bien varios objetos concatenados/delimitados por líneas. Si se elige esta opción en un conjunto de datos de salida, la actividad de copia genera un único archivo JSON con cada objeto por línea (delimitado por líneas).

    * **ejemplo de JSON de objeto único**

        ```json
        {
            "time": "2015-04-29T07:12:20.9100000Z",
            "callingimsi": "466920403025604",
            "callingnum1": "678948008",
            "callingnum2": "567834760",
            "switch1": "China",
            "switch2": "Germany"
        }
        ```

    * **ejemplo de JSON delimitado por líneas**

        ```json
        {"time":"2015-04-29T07:12:20.9100000Z","callingimsi":"466920403025604","callingnum1":"678948008","callingnum2":"567834760","switch1":"China","switch2":"Germany"}
        {"time":"2015-04-29T07:13:21.0220000Z","callingimsi":"466922202613463","callingnum1":"123436380","callingnum2":"789037573","switch1":"US","switch2":"UK"}
        {"time":"2015-04-29T07:13:21.4370000Z","callingimsi":"466923101048691","callingnum1":"678901578","callingnum2":"345626404","switch1":"Germany","switch2":"UK"}
        ```

    * **ejemplo de JSON concatenado**

        ```json
        {
            "time": "2015-04-29T07:12:20.9100000Z",
            "callingimsi": "466920403025604",
            "callingnum1": "678948008",
            "callingnum2": "567834760",
            "switch1": "China",
            "switch2": "Germany"
        }
        {
            "time": "2015-04-29T07:13:21.0220000Z",
            "callingimsi": "466922202613463",
            "callingnum1": "123436380",
            "callingnum2": "789037573",
            "switch1": "US",
            "switch2": "UK"
        }
        {
            "time": "2015-04-29T07:13:21.4370000Z",
            "callingimsi": "466923101048691",
            "callingnum1": "678901578",
            "callingnum2": "345626404",
            "switch1": "Germany",
            "switch2": "UK"
        }
        ```

- **Tipo II: arrayOfObjects**

    Cada archivo contiene una matriz de objetos.

    ```json
    [
        {
            "time": "2015-04-29T07:12:20.9100000Z",
            "callingimsi": "466920403025604",
            "callingnum1": "678948008",
            "callingnum2": "567834760",
            "switch1": "China",
            "switch2": "Germany"
        },
        {
            "time": "2015-04-29T07:13:21.0220000Z",
            "callingimsi": "466922202613463",
            "callingnum1": "123436380",
            "callingnum2": "789037573",
            "switch1": "US",
            "switch2": "UK"
        },
        {
            "time": "2015-04-29T07:13:21.4370000Z",
            "callingimsi": "466923101048691",
            "callingnum1": "678901578",
            "callingnum2": "345626404",
            "switch1": "Germany",
            "switch2": "UK"
        }
    ]
    ```

### <a name="jsonformat-example"></a>Ejemplo de JsonFormat

**Caso 1: Copia de datos desde archivos JSON**

**Ejemplo 1: extracción de datos de objeto y matriz**

En esta ejemplo, se espera un objeto JSON de raíz que se asigna al registro individual en resultado tabulares. Si tiene un archivo JSON con el siguiente contenido:

```json
{
    "id": "ed0e4960-d9c5-11e6-85dc-d7996816aad3",
    "context": {
        "device": {
            "type": "PC"
        },
        "custom": {
            "dimensions": [
                {
                    "TargetResourceType": "Microsoft.Compute/virtualMachines"
                },
                {
                    "ResourceManagementProcessRunId": "827f8aaa-ab72-437c-ba48-d8917a7336a3"
                },
                {
                    "OccurrenceTime": "1/13/2017 11:24:37 AM"
                }
            ]
        }
    }
}
```

y quiere copiarlo en una tabla de SQL de Azure con el formato siguiente extrayendo datos tanto de los objetos como de la matriz:

| ID | deviceType | targetResourceType | resourceManagementProcessRunId | occurrenceTime |
| --- | --- | --- | --- | --- |
| ed0e4960-d9c5-11e6-85dc-d7996816aad3 | PC | Microsoft.Compute/virtualMachines | 827f8aaa-ab72-437c-ba48-d8917a7336a3 | 1/13/2017 11:24:37 AM |

El conjunto de datos de entrada con el tipo **JsonFormat** se define de la siguiente manera: (definición parcial en la que solo se ilustran las secciones relevantes). Más concretamente:

- La sección `structure` permite definir los nombres personalizados de columna y el tipo de datos correspondiente mientras los convierte a datos tabulares. Esta sección es **opcional** a menos que necesite realizar una asignación de columnas. Consulte la sección [Asignación de columnas de conjunto de datos de origen a columnas del conjunto de datos de destino](copy-activity-schema-and-type-mapping.md) para más información.
- `jsonPathDefinition` especifica la ruta de acceso JSON para cada columna que indica de dónde se deben extraer los datos. Para copiar datos de la matriz, puede usar `array[x].property` para extraer el valor de la propiedad dada del objeto `xth`, o puede usar `array[*].property` para buscar el valor de cualquier objeto que contenga dicha propiedad.

```json
"properties": {
    "structure": [
        {
            "name": "id",
            "type": "String"
        },
        {
            "name": "deviceType",
            "type": "String"
        },
        {
            "name": "targetResourceType",
            "type": "String"
        },
        {
            "name": "resourceManagementProcessRunId",
            "type": "String"
        },
        {
            "name": "occurrenceTime",
            "type": "DateTime"
        }
    ],
    "typeProperties": {
        "folderPath": "mycontainer/myfolder",
        "format": {
            "type": "JsonFormat",
            "filePattern": "setOfObjects",
            "jsonPathDefinition": {"id": "$.id", "deviceType": "$.context.device.type", "targetResourceType": "$.context.custom.dimensions[0].TargetResourceType", "resourceManagementProcessRunId": "$.context.custom.dimensions[1].ResourceManagementProcessRunId", "occurrenceTime": " $.context.custom.dimensions[2].OccurrenceTime"}
        }
    }
}
```

**Ejemplo 2: aplicación cruzada en varios objetos del mismo patrón de matriz**

En este ejemplo, se espera transformar un objeto JSON de raíz en varios registros en resultado tabular. Si tiene un archivo JSON con el siguiente contenido:

```json
{
    "ordernumber": "01",
    "orderdate": "20170122",
    "orderlines": [
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
    "city": [ { "sanmateo": "No 1" } ]
}
```

y desea copiarlo en una tabla de Azure SQL del formato siguiente, puede hacerlo mediante el acoplamiento de los datos dentro de la matriz y la combinación cruzada con la información de la raíz habitual:

| `ordernumber` | `orderdate` | `order_pd` | `order_price` | `city` |
| --- | --- | --- | --- | --- |
| 01 | 20170122 | P1 | 23 | `[{"sanmateo":"No 1"}]` |
| 01 | 20170122 | P2 | 13 | `[{"sanmateo":"No 1"}]` |
| 01 | 20170122 | P3 | 231 | `[{"sanmateo":"No 1"}]` |


El conjunto de datos de entrada con el tipo **JsonFormat** se define de la siguiente manera: (definición parcial en la que solo se ilustran las secciones relevantes). Más concretamente:

- La sección `structure` permite definir los nombres personalizados de columna y el tipo de datos correspondiente mientras los convierte a datos tabulares. Esta sección es **opcional** a menos que necesite realizar una asignación de columnas. Consulte la sección [Asignación de columnas de conjunto de datos de origen a columnas del conjunto de datos de destino](copy-activity-schema-and-type-mapping.md) para más información.
- `jsonNodeReference` indica la iteración y extracción de datos de los objetos con el mismo patrón en la **matriz** `orderlines`.
- `jsonPathDefinition` especifica la ruta de acceso JSON para cada columna que indica de dónde se deben extraer los datos. En este ejemplo, `ordernumber`, `orderdate` y `city` están bajo el objeto raíz con la ruta de acceso JSON que empieza por `$.`, mientras que `order_pd` y `order_price` se definen con la ruta de acceso que se obtiene del elemento de matriz sin `$.`.

```json
"properties": {
    "structure": [
        {
            "name": "ordernumber",
            "type": "String"
        },
        {
            "name": "orderdate",
            "type": "String"
        },
        {
            "name": "order_pd",
            "type": "String"
        },
        {
            "name": "order_price",
            "type": "Int64"
        },
        {
            "name": "city",
            "type": "String"
        }
    ],
    "typeProperties": {
        "folderPath": "mycontainer/myfolder",
        "format": {
            "type": "JsonFormat",
            "filePattern": "setOfObjects",
            "jsonNodeReference": "$.orderlines",
            "jsonPathDefinition": {"ordernumber": "$.ordernumber", "orderdate": "$.orderdate", "order_pd": "prod", "order_price": "price", "city": " $.city"}
        }
    }
}
```

**Tenga en cuenta los siguientes puntos:**

* Si no se definen la `structure` y `jsonPathDefinition` en el conjunto de datos de Data Factory, la actividad de copia detecta el esquema del primer objeto y acopla el objeto en su conjunto.
* Si la entrada JSON tiene una matriz, la actividad de copia convierte de forma predeterminada el valor de toda la matriz en una cadena. Puede elegir extraer los datos de esta mediante `jsonNodeReference` o `jsonPathDefinition`, u omitir la extracción no especificándolo en `jsonPathDefinition`.
* Si hay algún nombre duplicado en el mismo nivel, la actividad de copia elige el último.
* Los nombres de propiedad distinguen entre mayúsculas y minúsculas. Dos propiedades con el mismo nombre, pero con distintas mayúsculas y minúsculas se consideran propiedades independientes.

**Caso 2: Escritura de datos en el archivo JSON**

Si tiene la siguiente tabla en SQL Database:

| ID | order_date | order_price | order_by |
| --- | --- | --- | --- |
| 1 | 20170119 | 2000 | David |
| 2 | 20170120 | 3500 | Patrick |
| 3 | 20170121 | 4000 | Jason |

y para cada registro, espera escribir en un objeto JSON con el formato siguiente:

```json
{
    "id": "1",
    "order": {
        "date": "20170119",
        "price": 2000,
        "customer": "David"
    }
}
```

El conjunto de datos de salida con el tipo **JsonFormat** se define de la siguiente manera: (definición parcial en la que solo se ilustran las secciones relevantes). Más concretamente, la sección `structure` permite definir los nombres de propiedad personalizados en el archivo de destino y `nestingSeparator` (el valor predeterminado es ".") se usa para identificar el nivel de anidamiento del nombre. Esta sección es **opcional** a menos que desee cambiar el nombre de la propiedad comparándolo con el nombre de la columna de origen, o anidar algunas de las propiedades.

```json
"properties": {
    "structure": [
        {
            "name": "id",
            "type": "String"
        },
        {
            "name": "order.date",
            "type": "String"
        },
        {
            "name": "order.price",
            "type": "Int64"
        },
        {
            "name": "order.customer",
            "type": "String"
        }
    ],
    "typeProperties": {
        "folderPath": "mycontainer/myfolder",
        "format": {
            "type": "JsonFormat"
        }
    }
}
```

## <a name="parquet-format-legacy"></a><a name="parquet-format"></a> Formato Parquet (heredado)

>[!NOTE]
>Obtenga información sobre el nuevo modelo en el artículo [Formato Parquet](format-parquet.md). Todavía se admiten las siguientes configuraciones en el conjunto de datos basado en archivo, porque cuentan con compatibilidad con versiones anteriores. Se sugiere usar el nuevo modelo de aquí en adelante.

Si quiere analizar los archivos Parquet o escribir los datos en formato Parquet, establezca la propiedad `format` `type` en **ParquetFormat**. No es preciso especificar propiedades en la sección Format de la sección typeProperties. Ejemplo:

```json
"format":
{
    "type": "ParquetFormat"
}
```

Tenga en cuenta los siguientes puntos:

* No se admiten tipos de datos complejos (MAP, LIST).
* No se admiten espacios en blanco en el nombre de columna.
* El archivo Parquet tiene las siguientes opciones relacionadas con la compresión: NONE, SNAPPY, GZIP y LZO. Data Factory admite la lectura de datos de archivos Parquet en todos estos formatos comprimidos excepto LZO. Este usa el códec de compresión de los metadatos para leer los datos. Sin embargo, al escribir en un archivo Parquet, Data Factory elige SNAPPY que es el valor predeterminado para Parquet. Por el momento, no hay ninguna opción para invalidar este comportamiento.

> [!IMPORTANT]
> En el caso de las copias autorizadas por el entorno de ejecución de integración (IR) autohospedado (por ejemplo, entre almacenes de datos locales y almacenes de datos en la nube), si no va a copiar archivos Parquet **tal y como están**, tendrá que instalar **JRE (Java Runtime Environment) 8 de 64 bits u OpenJDK** en la máquina de IR. Consulte el párrafo siguiente para más información.

En el caso de las copias que se ejecutan en el IR autohospedado con la serialización o deserialización de archivos Parquet, para encontrar el entorno de ejecución de Java, ADF consulta primero el Registro *`(SOFTWARE\JavaSoft\Java Runtime Environment\{Current Version}\JavaHome)`* de JRE; si no lo encuentra, comprueba la variable del sistema *`JAVA_HOME`* de OpenJDK.

- **Para usar JRE**: el IR de 64 bits necesita JRE de 64 bits. Puede encontrarlo [aquí](https://go.microsoft.com/fwlink/?LinkId=808605).
- **Para usar OpenJDK**: se admite desde la versión 3.13 de IR. Empaquete jvm.dll con todos los demás ensamblados de OpenJDK necesarios en la máquina del IR autohospedado y establezca la variable de entorno del sistema JAVA_HOME en el valor que corresponda.

>[!TIP]
>Si copia datos desde o hacia Parquet mediante Integration Runtime autohospedado y recibe un error que indica que "Se produjo un error al invocar Java, mensaje: **Espacio en el montón java.lang.OutOfMemoryError:Java**", puede agregar una variable de entorno `_JAVA_OPTIONS` en la máquina que hospeda IR autohospedado para ajustar el tamaño del montón mínimo y máximo para JVM a fin de facilitar dicha copia y, a continuación, volver a ejecutar la canalización.

![Establecimiento del tamaño del montón JVM en IR autohospedado](./media/supported-file-formats-and-compression-codecs/set-jvm-heap-size-on-selfhosted-ir.png)

Ejemplo: establecimiento de la variable `_JAVA_OPTIONS` con el valor `-Xms256m -Xmx16g`. La marca `Xms` especifica el grupo de asignación de memoria inicial para una máquina virtual Java (JVM), mientras que `Xmx` especifica el grupo de asignación de memoria máxima. Esto significa que JVM se iniciará con la cantidad de memoria `Xms` y podrá utilizar `Xmx` como máximo. De forma predeterminada, ADF usa 64 MB como mínimo y 1 GB como máximo.

### <a name="data-type-mapping-for-parquet-files"></a>Asignación de tipos de datos para archivos PARQUET

| Tipo de datos provisionales de Data Factory | Tipo primitivo PARQUET | Tipo original PARQUET (deserializar) | Tipo original PARQUET (serializar) |
|:--- |:--- |:--- |:--- |
| Boolean | Boolean | N/D | N/D |
| SByte | Int32 | Int8 | Int8 |
| Byte | Int32 | UInt8 | Int16 |
| Int16 | Int32 | Int16 | Int16 |
| UInt16 | Int32 | UInt16 | Int32 |
| Int32 | Int32 | Int32 | Int32 |
| UInt32 | Int64 | UInt32 | Int64 |
| Int64 | Int64 | Int64 | Int64 |
| UInt64 | Int64/binario | UInt64 | Decimal |
| Single | Float | N/D | N/D |
| Double | Double | N/D | N/D |
| Decimal | Binary | Decimal | Decimal |
| String | Binary | Utf8 | Utf8 |
| DateTime | Int96 | N/D | N/D |
| TimeSpan | Int96 | N/D | N/D |
| DateTimeOffset | Int96 | N/D | N/D |
| ByteArray | Binary | N/D | N/D |
| Guid | Binary | Utf8 | Utf8 |
| Char | Binary | Utf8 | Utf8 |
| CharArray | No compatible | N/D | N/D |

## <a name="orc-format-legacy"></a><a name="orc-format"></a> Formato ORC (heredado)

>[!NOTE]
>Obtenga información sobre el nuevo modelo en el artículo [Formato ORC](format-orc.md). Todavía se admiten las siguientes configuraciones en el conjunto de datos basado en archivo, porque cuentan con compatibilidad con versiones anteriores. Se sugiere usar el nuevo modelo de aquí en adelante.

Si quiere analizar los archivos ORC o escribir los datos en formato ORC, establezca la propiedad `format` `type` en **OrcFormat**. No es preciso especificar propiedades en la sección Format de la sección typeProperties. Ejemplo:

```json
"format":
{
    "type": "OrcFormat"
}
```

Tenga en cuenta los siguientes puntos:

* No se admiten tipos de datos complejos (STRUCT, MAP, LIST, UNION).
* No se admiten espacios en blanco en el nombre de columna.
* El archivo ORC tiene tres [opciones relacionadas con la compresión](https://hortonworks.com/blog/orcfile-in-hdp-2-better-compression-better-performance/): NONE, ZLIB Y SNAPPY. Data Factory admite la lectura de datos del archivo ORC en cualquiera de los formatos comprimidos. Se utiliza el códec de compresión en los metadatos para leer los datos. Sin embargo, al escribir en un archivo ORC, Data Factory elige ZLIB que es el valor predeterminado para ORC. Por el momento, no hay ninguna opción para invalidar este comportamiento.

> [!IMPORTANT]
> En el caso de las copias autorizadas por el entorno de ejecución de integración (IR) autohospedado (por ejemplo, entre almacenes de datos locales y almacenes de datos en la nube), si no va a copiar archivos ORC **tal y como están**, tendrá que instalar **JRE (Java Runtime Environment) 8 de 64 bits u OpenJDK** en la máquina de IR. Consulte el párrafo siguiente para más información.

En el caso de las copias que se ejecutan en el IR autohospedado con la serialización o deserialización de archivos ORC, para encontrar el entorno de ejecución de Java, ADF consulta primero el Registro *`(SOFTWARE\JavaSoft\Java Runtime Environment\{Current Version}\JavaHome)`* de JRE; si no lo encuentra, comprueba la variable del sistema *`JAVA_HOME`* de OpenJDK.

- **Para usar JRE**: el IR de 64 bits necesita JRE de 64 bits. Puede encontrarlo [aquí](https://go.microsoft.com/fwlink/?LinkId=808605).
- **Para usar OpenJDK**: se admite desde la versión 3.13 de IR. Empaquete jvm.dll con todos los demás ensamblados de OpenJDK necesarios en la máquina del IR autohospedado y establezca la variable de entorno del sistema JAVA_HOME en el valor que corresponda.

### <a name="data-type-mapping-for-orc-files"></a>Asignación de tipos de datos para archivos ORC

| Tipo de datos provisionales de Data Factory | Tipos ORC |
|:--- |:--- |
| Boolean | Boolean |
| SByte | Byte |
| Byte | Short |
| Int16 | Short |
| UInt16 | Int |
| Int32 | Int |
| UInt32 | long |
| Int64 | long |
| UInt64 | String |
| Single | Float |
| Double | Double |
| Decimal | Decimal |
| String | String |
| DateTime | Timestamp |
| DateTimeOffset | Timestamp |
| TimeSpan | Timestamp |
| ByteArray | Binary |
| Guid | String |
| Char | Char(1) |

## <a name="avro-format-legacy"></a><a name="avro-format"></a> Formato AVRO (heredado)

>[!NOTE]
>Obtenga información sobre el nuevo modelo en el artículo [Formato Avro](format-avro.md). Todavía se admiten las siguientes configuraciones en el conjunto de datos basado en archivo, porque cuentan con compatibilidad con versiones anteriores. Se sugiere usar el nuevo modelo de aquí en adelante.

Si quiere analizar los archivos Avro o escribir los datos en formato Avro, establezca la propiedad `format` `type` en **AvroFormat**. No es preciso especificar propiedades en la sección Format de la sección typeProperties. Ejemplo:

```json
"format":
{
    "type": "AvroFormat",
}
```

Para usar el formato Avro en una tabla de Hive, puede consultar [Tutorial de Apache Hive](https://cwiki.apache.org/confluence/display/Hive/AvroSerDe).

Tenga en cuenta los siguientes puntos:

* No se admiten [tipos de datos complejos](https://avro.apache.org/docs/current/spec.html#schema_complex) (registros, enumeraciones, matrices, asignaciones, uniones y fijos).

## <a name="compression-support-legacy"></a><a name="compression-support"></a> Compatibilidad con la compresión (heredado)

Azure Data Factory admite datos de compresión y descompresión durante la copia. Cuando se especifica la propiedad `compression` en un conjunto de datos de entrada, la actividad de copia lee los datos comprimidos del origen y los descomprime; y cuando se especifica la propiedad en un conjunto de datos de salida, la actividad de copia comprime los datos y luego los escribe en el receptor. Estos son algunos escenarios de ejemplo:

* Leer datos comprimidos GZIP de un blob de Azure, descomprimirlos y escribir los datos de resultado en Azure SQL Database. Debe definir el conjunto de datos de Azure Blob de entrada con la propiedad `compression` `type` como GZIP.
* Leer datos de un archivo de texto sin formato del sistema de archivos local, comprimirlos con formato GZip y escribir los datos comprimidos en un blob de Azure. Debe definir el conjunto de datos de Azure Blob de salida con la propiedad `compression` `type` como GZIP.
* Leer el archivo .zip del servidor FTP, descomprimirlo para obtener los archivos que contiene y colocar dichos archivos en Azure Data Lake Store. Debe definir un conjunto de datos de FTP de entrada con la propiedad `compression` `type` como ZipDeflate.
* Leer datos comprimidos con GZIP de un blob de Azure, descomprimirlos, comprimirlos con BZIP2 y escribir los datos de resultado en un blob de Azure. Debe definir el conjunto de datos de Azure Blob de entrada con `compression` `type` establecido en GZIP y el conjunto de datos de salida con `compression` `type` establecido en BZIP2.

Para especificar la compresión para un conjunto de datos, use la propiedad **compression** del conjunto de datos JSON como en el ejemplo siguiente:

```json
{
    "name": "AzureBlobDataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": {
            "referenceName": "StorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "fileName": "pagecounts.csv.gz",
            "folderPath": "compression/file/",
            "format": {
                "type": "TextFormat"
            },
            "compression": {
                "type": "GZip",
                "level": "Optimal"
            }
        }
    }
}
```

La sección **compression** tiene dos propiedades:

* **Type**: el códec de compresión, que puede ser **GZIP**, **Deflate** o **BZIP2** o **ZipDeflate**. Tenga en cuenta que cuando utilice la actividad de copia para descomprimir archivos ZipDeflate y escribir en el almacén de datos de receptores basado en archivos, los archivos se extraerán a la carpeta `<path specified in dataset>/<folder named as source zip file>/`.
* **Level**: la relación de compresión, que puede ser **Optimal** o **Fastest**.

  * **Fastest**: la operación de compresión debe completarse tan pronto como sea posible, incluso si el archivo resultante no se comprime de forma óptima.
  * **Optimal**: la operación de compresión se debe comprimir óptimamente, incluso si tarda más tiempo en completarse.

    Para más información, consulte el tema [Nivel de compresión](/dotnet/api/system.io.compression.compressionlevel) .

> [!NOTE]
> La configuración de la compresión no se admite para los datos con los formatos **AvroFormat**, **OrcFormat** o **ParquetFormat**. Al leer archivos en estos formatos, Data Factory detecta y usa el códec de compresión en los metadatos. Sin embargo, al escribir en archivos en estos formatos, Data Factory elige el códec de compresión predeterminado para ese formato. Por ejemplo, ZLIB en el caso de OrcFormat y SNAPPY en el caso de ParquetFormat.

## <a name="unsupported-file-types-and-compression-formats"></a>Formatos de compresión y tipos de archivo no admitidos

Puede usar las características de extensibilidad de Azure Data Factory para transformar los archivos que no son compatibles.
Las dos opciones incluyen Azure Functions y las tareas personalizadas con Azure Batch.

Puede ver un ejemplo que usa una función de Azure para [extraer el contenido de un archivo tar](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV2/UntarAzureFilesWithAzureFunction). Para más información, consulte la [actividad de Azure Functions](./control-flow-azure-function-activity.md).

También puede crear esta funcionalidad a través de una actividad de DotNet personalizada. [Aquí](./transform-data-using-dotnet-custom-activity.md) hay disponible más información

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre los formatos de archivo y las compresiones admitidos más recientes en [Compresiones y formatos de archivo compatibles](supported-file-formats-and-compression-codecs.md).