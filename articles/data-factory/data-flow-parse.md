---
title: Transformación de análisis de datos en el flujo de datos de asignación
description: Análisis de documentos de columnas insertadas
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.date: 05/10/2021
ms.openlocfilehash: 7a01d2d17a4c98656588530f5b288c6a69b8a206
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109734173"
---
# <a name="parse-transformation-in-mapping-data-flow"></a>Transformación de análisis en el flujo de datos de asignación

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Use la transformación de análisis para analizar las columnas de los datos del documento. Los tipos de documentos insertados admitidos actualmente que se pueden analizar son JSON, XML y texto delimitado.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RWykdO]

## <a name="configuration"></a>Configuración

En el panel de configuración de la transformación de análisis, primero se selecciona el tipo de datos contenidos en las columnas que desea analizar en línea. La transformación de análisis también contiene los siguientes valores de configuración.

![Configuración de análisis](media/data-flow/data-flow-parse-1.png "Analizar")

### <a name="column"></a>Columna

De forma similar a las columnas derivadas y los agregados, aquí es donde modificará una columna de salida, para lo cual deberá seleccionarla en el selector desplegable. También puede escribir aquí el nombre de una nueva columna. ADF almacenará los datos de origen analizados en esta columna. En la mayoría de los casos, será aconsejable definir una nueva columna que analice el campo de documento insertado de entrada.

### <a name="expression"></a>Expression

Use el generador de expresiones para establecer el origen del análisis. Esto puede ser tan sencillo como seleccionar la columna de origen con los datos independientes que desea analizar, o bien puede crear expresiones complejas para analizarlas.

#### <a name="example-expressions"></a>Expresiones de ejemplo

* Datos de la cadena de origen: ```chrome|steel|plastic```
  * Expresión: ```(desc1 as string, desc2 as string, desc3 as string)```

* Datos JSON de origen: ```{"ts":1409318650332,"userId":"309","sessionId":1879,"page":"NextSong","auth":"Logged In","method":"PUT","status":200,"level":"free","itemInSession":2,"registration":1384448}```
  * Expresión: ```(level as string, registration as long)```

* Datos XML de origen: ```<Customers><Customer>122</Customer><CompanyName>Great Lakes Food Market</CompanyName></Customers>```
  * Expresión: ```(Customers as (Customer as integer, CompanyName as string))```

### <a name="output-column-type"></a>Tipo de columna de salida

Aquí es donde va a configurar el esquema de salida de destino a partir del análisis que se escribirá en una sola columna.

![Ejemplo de análisis](media/data-flow/data-flow-parse-2.png "Ejemplo de análisis")

En este ejemplo, hemos definido el análisis del campo de entrada "jsonString", que es texto sin formato, pero con formato de estructura JSON. Vamos a almacenar los resultados analizados como JSON en una nueva columna denominada "json" con este esquema:

```(trade as boolean, customers as string[])```

Consulte la pestaña de inspección y la vista previa de los datos para comprobar que la salida se ha asignado correctamente.

## <a name="examples"></a>Ejemplos

```
source(output(
        name as string,
        location as string,
        satellites as string[],
        goods as (trade as boolean, customers as string[], orders as (orderId as string, orderTotal as double, shipped as (orderItems as (itemName as string, itemQty as string)[]))[])
    ),
    allowSchemaDrift: true,
    validateSchema: false,
    ignoreNoFilesFound: false,
    documentForm: 'documentPerLine') ~> JsonSource
source(output(
        movieId as string,
        title as string,
        genres as string
    ),
    allowSchemaDrift: true,
    validateSchema: false,
    ignoreNoFilesFound: false) ~> CsvSource
JsonSource derive(jsonString = toString(goods)) ~> StringifyJson
StringifyJson parse(json = jsonString ? (trade as boolean,
        customers as string[]),
    format: 'json',
    documentForm: 'arrayOfDocuments') ~> ParseJson
CsvSource derive(csvString = 'Id|name|year\n\'1\'|\'test1\'|\'1999\'') ~> CsvString
CsvString parse(csv = csvString ? (id as integer,
        name as string,
        year as string),
    format: 'delimited',
    columnNamesAsHeader: true,
    columnDelimiter: '|',
    nullValue: '',
    documentForm: 'documentPerLine') ~> ParseCsv
ParseJson select(mapColumn(
        jsonString,
        json
    ),
    skipDuplicateMapInputs: true,
    skipDuplicateMapOutputs: true) ~> KeepStringAndParsedJson
ParseCsv select(mapColumn(
        csvString,
        csv
    ),
    skipDuplicateMapInputs: true,
    skipDuplicateMapOutputs: true) ~> KeepStringAndParsedCsv
```

## <a name="data-flow-script"></a>Script de flujo de datos

### <a name="syntax"></a>Sintaxis

### <a name="examples"></a>Ejemplos

```
parse(json = jsonString ? (trade as boolean,
                                customers as string[]),
                format: 'json|XML|delimited',
                documentForm: 'singleDocument') ~> ParseJson

parse(csv = csvString ? (id as integer,
                                name as string,
                                year as string),
                format: 'delimited',
                columnNamesAsHeader: true,
                columnDelimiter: '|',
                nullValue: '',
                documentForm: 'documentPerLine') ~> ParseCsv
```    

## <a name="next-steps"></a>Pasos siguientes

* Use la [transformación Aplanar](data-flow-flatten.md) para dinamizar las filas en columnas.
* Use la [transformación de columna derivada](data-flow-derived-column.md) para dinamizar columnas en filas.
