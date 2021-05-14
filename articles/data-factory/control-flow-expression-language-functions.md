---
title: Expresiones y funciones en Azure Data Factory
description: En este artículo se proporciona información sobre las expresiones y las funciones que se pueden usar al crear entidades de Data Factory.
author: minhe-msft
ms.author: hemin
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 04/28/2021
ms.openlocfilehash: 275c77107faf8fd639d714b92828ab8efe623f26
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108164910"
---
# <a name="expressions-and-functions-in-azure-data-factory"></a>Expresiones y funciones de Azure Data Factory

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-functions-variables.md)
> * [Versión actual](control-flow-expression-language-functions.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se proporciona información detallada sobre las expresiones y las funciones compatibles con Azure Data Factory. 

## <a name="expressions"></a>Expresiones

Los valores JSON de la definición pueden ser literales o expresiones que se evalúan en tiempo de ejecución. Por ejemplo:  
  
```json
"name": "value"
```

 or  
  
```json
"name": "@pipeline().parameters.password"
```

Las expresiones pueden aparecer en cualquier lugar de un valor de cadena JSON y devolver siempre otro valor JSON. Si un valor JSON es una expresión, se extrae el cuerpo de la expresión quitando el signo arroba (\@). Si se necesita una cadena literal que empiece por \@, debe convertirse con \@\@. Los ejemplos siguientes muestran cómo se evalúan las expresiones.  
  
|Valor JSON|Resultado|  
|----------------|------------|  
|"parameters"|Se devuelven los caracteres de "parameters".|  
|"parameters[1]"|Se devuelven los caracteres de "parameters[1]".|  
|"\@\@"|Se devuelve una cadena de 1 carácter que contiene·"\@".|  
|" \@"|Se devuelve una cadena de 2 caracteres que contiene "\@".|  
  
 Las expresiones también pueden aparecer dentro de las cadenas mediante una característica llamada *interpolación de cadenas*, donde las expresiones se ajustan en `@{ ... }`. Por ejemplo: `"name" : "First Name: @{pipeline().parameters.firstName} Last Name: @{pipeline().parameters.lastName}"`  
  
 Con la interpolación de cadena, el resultado siempre es una cadena. Supongamos que se ha definido `myNumber` como `42` y `myString` como `foo`:  
  
|Valor JSON|Resultado|  
|----------------|------------|  
|"\@pipeline().parameters.myString"| Devuelve `foo` como una cadena.|  
|"\@{pipeline().parameters.myString}"| Devuelve `foo` como una cadena.|  
|"\@pipeline().parameters.myNumber"| Devuelve `42` como un *número*.|  
|"\@{pipeline().parameters.myNumber}"| Devuelve `42` como una *cadena*.|  
|"Answer is: @{pipeline().parameters.myNumber}"| Devuelve la cadena `Answer is: 42`.|  
|"\@concat('Answer is: ', string(pipeline().parameters.myNumber))"| Devuelve la cadena `Answer is: 42`.|  
|"Answer is: \@\@{pipeline().parameters.myNumber}"| Devuelve la cadena `Answer is: @{pipeline().parameters.myNumber}`.|  

## <a name="examples"></a>Ejemplos

### <a name="complex-expression-example"></a>Ejemplo de expresión compleja
En el ejemplo siguiente se muestra un ejemplo complejo que hace referencia a un subcampo profundo de la salida de la actividad. Para hacer referencia a un parámetro de canalización que se evalúa como un subcampo, use la sintaxis [] en lugar del operador punto (.) (como en el caso de subfield1 y subfield2)

`@activity('*activityName*').output.*subfield1*.*subfield2*[pipeline().parameters.*subfield3*].*subfield4*`

### <a name="dynamic-content-editor"></a>Editor de contenido dinámico

El editor de contenido dinámico convierte automáticamente los caracteres de escape en el contenido cuando finaliza la edición. Por ejemplo, el contenido siguiente del editor de contenido es una interpolación de cadenas con dos funciones de expresión. 

```json
{ 
  "type": "@{if(equals(1, 2), 'Blob', 'Table' )}",
  "name": "@{toUpper('myData')}"
}
```

El editor de contenido dinámico convierte el contenido anterior en la expresión `"{ \n  \"type\": \"@{if(equals(1, 2), 'Blob', 'Table' )}\",\n  \"name\": \"@{toUpper('myData')}\"\n}"`. El resultado de esta expresión es la cadena de formato JSON que se muestra a continuación.

```json
{
  "type": "Table",
  "name": "MYDATA"
}
```

### <a name="a-dataset-with-a-parameter"></a>Un conjunto de datos con un parámetro
En el ejemplo siguiente, BlobDataset toma un parámetro llamado **path**. Su valor se usa para establecer un valor para la propiedad **folderPath** mediante la expresión: `dataset().path`. 

```json
{
    "name": "BlobDataset",
    "properties": {
        "type": "AzureBlob",
        "typeProperties": {
            "folderPath": "@dataset().path"
        },
        "linkedServiceName": {
            "referenceName": "AzureStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "parameters": {
            "path": {
                "type": "String"
            }
        }
    }
}
```

### <a name="a-pipeline-with-a-parameter"></a>Una canalización con un parámetro
En el ejemplo siguiente, la canalización toma los parámetros **inputPath** y **outputPath**. El valor de **path** para el conjunto de datos del blob con parámetros se establece mediante el uso de los valores de estos parámetros. La sintaxis utilizada aquí es: `pipeline().parameters.parametername`. 

```json
{
    "name": "Adfv2QuickStartPipeline",
    "properties": {
        "activities": [
            {
                "name": "CopyFromBlobToBlob",
                "type": "Copy",
                "inputs": [
                    {
                        "referenceName": "BlobDataset",
                        "parameters": {
                            "path": "@pipeline().parameters.inputPath"
                        },
                        "type": "DatasetReference"
                    }
                ],
                "outputs": [
                    {
                        "referenceName": "BlobDataset",
                        "parameters": {
                            "path": "@pipeline().parameters.outputPath"
                        },
                        "type": "DatasetReference"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                }
            }
        ],
        "parameters": {
            "inputPath": {
                "type": "String"
            },
            "outputPath": {
                "type": "String"
            }
        }
    }
}
```

### <a name="replacing-special-characters"></a>Reemplazo de caracteres especiales

El editor de contenido dinámico convierte automáticamente los caracteres de escape, como las comillas dobles y la barra diagonal inversa, en el contenido cuando finaliza la edición. Esto causa problemas si quiere reemplazar el salto de línea o la tabulación mediante **\n** o **\t** en la función replace(). Puede editar el contenido dinámico en la vista de código para quitar el la barra diagonal inversa \ adicional de la expresión, o puede usar los siguientes pasos para reemplazar los caracteres especiales mediante el lenguaje de expresiones:

1. Codificación de URL con el valor original de cadena
1. Reemplace la cadena codificada con URL; por ejemplo, salto de línea (%0A), retorno de carro (%0D), tabulación horizontal (%09).
1. Descodificación de URL

Por ejemplo, si la variable *companyName* tiene un carácter de nueva línea en su valor, la expresión `@uriComponentToString(replace(uriComponent(variables('companyName')), '%0A', ''))` puede quitar el carácter de nueva línea. 

```json
Contoso-
Corporation
```

### <a name="escaping-single-quote-character"></a>Escape de un carácter de comilla simple

Las funciones de expresión usan comillas simples para los parámetros de valor de cadena. Use dos comillas simples para escapar un carácter ' en las funciones de cadena. Por ejemplo, la expresión `@concat('Baba', '''s ', 'book store')` devolverá el siguiente resultado.

```
Baba's book store
```

### <a name="tutorial"></a>Tutorial
En este [tutorial](https://azure.microsoft.com/mediahandler/files/resourcefiles/azure-data-factory-passing-parameters/Azure%20data%20Factory-Whitepaper-PassingParameters.pdf) se le guiará a lo largo del proceso de pasar parámetros entre una canalización y una actividad, así como entre actividades.

  
## <a name="functions"></a>Functions

Puede llamar a funciones dentro de expresiones. En las siguientes secciones se proporciona información sobre las funciones que se pueden usar en una expresión.  

## <a name="string-functions"></a>Funciones de cadena  

Para trabajar con cadenas, puede usar estas funciones de cadena y también algunas [funciones de colección](#collection-functions).
Las funciones de cadena solo funcionan en cadenas.

| Función de cadena | Tarea |
| --------------- | ---- |
| [concat](control-flow-expression-language-functions.md#concat) | Combina dos o más cadenas y devuelve la cadena combinada. |
| [endsWith](control-flow-expression-language-functions.md#endswith) | Comprueba si una cadena termina con la subcadena especificada. |
| [guid](control-flow-expression-language-functions.md#guid) | Genera un identificador único global (GUID) como una cadena. |
| [indexOf](control-flow-expression-language-functions.md#indexof) | Devuelve la posición inicial de una subcadena. |
| [lastIndexOf](control-flow-expression-language-functions.md#lastindexof) | Devuelve la posición inicial de la última repetición de una subcadena. |
| [replace](control-flow-expression-language-functions.md#replace) | Reemplaza una subcadena por la cadena especificada y devuelve la cadena actualizada. |
| [split](control-flow-expression-language-functions.md#split) | Devuelve una matriz que contiene subcadenas, separadas por comas, de una cadena mayor en función de un carácter delimitador especificado en la cadena original. |
| [startsWith](control-flow-expression-language-functions.md#startswith) | Comprueba si una cadena comienza con una subcadena especificada. |
| [substring](control-flow-expression-language-functions.md#substring) | Devuelve caracteres de una cadena, a partir de la posición especificada. |
| [toLower](control-flow-expression-language-functions.md#toLower) | Devuelve una cadena en formato de minúsculas. |
| [toUpper](control-flow-expression-language-functions.md#toUpper) | Devuelve una cadena en formato de mayúsculas. |
| [trim](control-flow-expression-language-functions.md#trim) | Quita el espacio en blanco inicial y final de una cadena y devuelve la cadena actualizada. |

## <a name="collection-functions"></a>Funciones de colección

Para trabajar con colecciones, por lo general matrices, cadenas y, en ocasiones, diccionarios, puede usar estas funciones de colección.

| Función de colección | Tarea |
| ------------------- | ---- |
| [contains](control-flow-expression-language-functions.md#contains) | Comprueba si una colección contiene un elemento específico. |
| [empty](control-flow-expression-language-functions.md#empty) | Comprueba si una colección está vacía. |
| [first](control-flow-expression-language-functions.md#first) | Devuelve el primer elemento de una colección. |
| [intersection](control-flow-expression-language-functions.md#intersection) | Devuelve una colección que tiene *solo* los elementos comunes en las colecciones especificadas. |
| [join](control-flow-expression-language-functions.md#join) | Devuelve una cadena que tiene *todos* los elementos de una matriz, separados por el carácter especificado. |
| [last](control-flow-expression-language-functions.md#last) | Devuelve el último elemento de una colección. |
| [length](control-flow-expression-language-functions.md#length) | Devuelve el número de elementos de una cadena o una matriz. |
| [skip](control-flow-expression-language-functions.md#skip) | Quita elementos del principio de una colección y devuelve *todos los demás* elementos. |
| [take](control-flow-expression-language-functions.md#take) | Devuelve elementos del principio de una colección. |
| [union](control-flow-expression-language-functions.md#union) | Devuelve una colección que tiene *todos* los elementos de las colecciones especificadas. | 

## <a name="logical-functions"></a>Funciones lógicas  

Estas funciones son útiles en las condiciones y se pueden usar para evaluar cualquier tipo de lógica.  
  
| Función de comparación lógica | Tarea |
| --------------------------- | ---- |
| [and](control-flow-expression-language-functions.md#and) | Comprueba si todas las expresiones son true. |
| [equals](control-flow-expression-language-functions.md#equals) | Comprueba si ambos valores son equivalentes. |
| [greater](control-flow-expression-language-functions.md#greater) | Comprueba si el primer valor es mayor que el segundo. |
| [greaterOrEquals](control-flow-expression-language-functions.md#greaterOrEquals) | Comprueba si el primer valor es mayor o igual que el segundo. |
| [if](control-flow-expression-language-functions.md#if) | Comprueba si una expresión es true o false. En función del resultado, devuelve un valor especificado. |
| [less](control-flow-expression-language-functions.md#less) | Comprueba si el primer valor es menor que el segundo. |
| [lessOrEquals](control-flow-expression-language-functions.md#lessOrEquals) | Compruebe si el primer valor es menor o igual que el segundo valor. |
| [not](control-flow-expression-language-functions.md#not) | Comprueba si una expresión es false. |
| [or](control-flow-expression-language-functions.md#or) | Comprueba si al menos una expresión es true. |
  
## <a name="conversion-functions"></a>Funciones de conversión  

 Estas funciones se utilizan para convertir en cada uno de los tipos nativos del idioma:  
-   string
-   integer
-   FLOAT
-   boolean
-   arrays
-   dictionaries

| Función de conversión | Tarea |
| ------------------- | ---- |
| [array](control-flow-expression-language-functions.md#array) | Devuelve una matriz a partir de una única entrada especificada. Para varias entradas, consulte [createArray](control-flow-expression-language-functions.md#createArray). |
| [base64](control-flow-expression-language-functions.md#base64) | Devuelve la versión de una cadena codificada en base64. |
| [base64ToBinary](control-flow-expression-language-functions.md#base64ToBinary) | Devuelve la versión binaria de una cadena codificada en base64. |
| [base64ToString](control-flow-expression-language-functions.md#base64ToString) | Devuelve la versión de cadena de una cadena codificada en base64. |
| [binary](control-flow-expression-language-functions.md#binary) | Devuelve la versión binaria de un valor de entrada. |
| [bool](control-flow-expression-language-functions.md#bool) | Devuelve la versión booleana de un valor de entrada. |
| [coalesce](control-flow-expression-language-functions.md#coalesce) | Devuelve el primer valor distinto de null de uno o más parámetros. |
| [createArray](control-flow-expression-language-functions.md#createArray) | Devuelve una matriz a partir de varias entradas. |
| [dataUri](control-flow-expression-language-functions.md#dataUri) | Devuelve el URI de datos de un valor de entrada. |
| [dataUriToBinary](control-flow-expression-language-functions.md#dataUriToBinary) | Devuelve la versión binaria de un URI de datos. |
| [dataUriToString](control-flow-expression-language-functions.md#dataUriToString) | Devuelve la versión de cadena de un URI de datos. |
| [decodeBase64](control-flow-expression-language-functions.md#decodeBase64) | Devuelve la versión de cadena de una cadena codificada en base64. |
| [decodeDataUri](control-flow-expression-language-functions.md#decodeDataUri) | Devuelve la versión binaria de un URI de datos. |
| [decodeUriComponent](control-flow-expression-language-functions.md#decodeUriComponent) | Devuelve una cadena que reemplaza los caracteres de escape por versiones descodificadas. |
| [encodeUriComponent](control-flow-expression-language-functions.md#encodeUriComponent) | Devuelve una cadena que reemplaza los caracteres no seguros de la dirección URL por caracteres de escape. |
| [float](control-flow-expression-language-functions.md#float) | Devuelve un número de punto flotante de un valor de entrada. |
| [int](control-flow-expression-language-functions.md#int) | Devuelve la versión como número entero de una cadena. |
| [json](control-flow-expression-language-functions.md#json) | Devuelve el valor o el objeto de tipo Notación de objetos JavaScript (JSON) de una cadena o XML. |
| [string](control-flow-expression-language-functions.md#string) | Devuelve la versión de cadena de un valor de entrada. |
| [uriComponent](control-flow-expression-language-functions.md#uriComponent) | Devuelve la versión codificada con el URI de un valor de entrada mediante la sustitución de los caracteres no seguros de la dirección URL por caracteres de escape. |
| [uriComponentToBinary](control-flow-expression-language-functions.md#uriComponentToBinary) | Devuelve la versión binaria de una cadena codificada con el URI. |
| [uriComponentToString](control-flow-expression-language-functions.md#uriComponentToString) | Devuelve la versión de cadena de una cadena codificada con el URI. |
| [xml](control-flow-expression-language-functions.md#xml) | Devuelve la versión XML de una cadena. |
| [xpath](control-flow-expression-language-functions.md#xpath) | Comprueba el código XML de los nodos o valores que coinciden con una expresión XPath (XML Path Language) y devuelve los nodos o valores coincidentes. |

## <a name="math-functions"></a>Funciones matemáticas  
 Estas funciones pueden utilizarse para ambos tipos de números: **enteros** y **flotantes**.  

| Función matemática | Tarea |
| ------------- | ---- |
| [add](control-flow-expression-language-functions.md#add) | Devuelve el resultado de sumar dos números. |
| [div](control-flow-expression-language-functions.md#div) | Devuelve el resultado de dividir dos números. |
| [max](control-flow-expression-language-functions.md#max) | Devuelve el valor más alto de un conjunto de números o una matriz. |
| [min](control-flow-expression-language-functions.md#min) | Devuelve el valor más bajo de un conjunto de números o una matriz. |
| [mod](control-flow-expression-language-functions.md#mod) | Devuelve el resto de dividir dos números. |
| [mul](control-flow-expression-language-functions.md#mul) | Devuelve el producto de multiplicar dos números. |
| [rand](control-flow-expression-language-functions.md#rand) | Devuelve un entero aleatorio desde un intervalo especificado. |
| [range](control-flow-expression-language-functions.md#range) | Devuelve una matriz de enteros que comienza en un entero especificado. |
| [sub](control-flow-expression-language-functions.md#sub) | Devuelve el resultado de restar el segundo número del primero. |
  
## <a name="date-functions"></a>Funciones de fecha  

| Función de fecha u hora | Tarea |
| --------------------- | ---- |
| [addDays](control-flow-expression-language-functions.md#addDays) | Agrega un número de días a una marca de tiempo. |
| [addHours](control-flow-expression-language-functions.md#addHours) | Agrega un número de horas a una marca de tiempo. |
| [addMinutes](control-flow-expression-language-functions.md#addMinutes) | Agrega un número de minutos a una marca de tiempo. |
| [addSeconds](control-flow-expression-language-functions.md#addSeconds) | Agrega un número de segundos a una marca de tiempo. |
| [addToTime](control-flow-expression-language-functions.md#addToTime) | Agrega un número de unidades de tiempo a una marca de tiempo. Consulte también [getFutureTime](control-flow-expression-language-functions.md#getFutureTime). |
| [convertFromUtc](control-flow-expression-language-functions.md#convertFromUtc) | Convierte una marca de tiempo del formato Hora Universal Coordinada (UTC) a la zona horaria de destino. |
| [convertTimeZone](control-flow-expression-language-functions.md#convertTimeZone) | Convierte una marca de tiempo de la zona horaria de origen a la zona horaria de destino. |
| [convertToUtc](control-flow-expression-language-functions.md#convertToUtc) | Convierte una marca de tiempo de la zona horaria de origen al formato Hora Universal Coordinada (UTC). |
| [dayOfMonth](control-flow-expression-language-functions.md#dayOfMonth) | Devuelve el día del componente de mes de una marca de tiempo. |
| [dayOfWeek](control-flow-expression-language-functions.md#dayOfWeek) | Devuelve el día del componente de semana de una marca de tiempo. |
| [dayOfYear](control-flow-expression-language-functions.md#dayOfYear) | Devuelve el día del componente de año de una marca de tiempo. |
| [formatDateTime](control-flow-expression-language-functions.md#formatDateTime) | Devuelve la marca de tiempo como cadena en formato opcional. |
| [getFutureTime](control-flow-expression-language-functions.md#getFutureTime) | Devuelve la marca de tiempo actual más las unidades de tiempo especificadas. Consulte también [addToTime](control-flow-expression-language-functions.md#addToTime). |
| [getPastTime](control-flow-expression-language-functions.md#getPastTime) | Devuelve la marca de tiempo actual menos las unidades de tiempo especificadas. Consulte también [subtractFromTime](control-flow-expression-language-functions.md#subtractFromTime). |
| [startOfDay](control-flow-expression-language-functions.md#startOfDay) | Devuelve el inicio del día de una marca de tiempo. |
| [startOfHour](control-flow-expression-language-functions.md#startOfHour) | Devuelve el inicio de la hora de una marca de tiempo. |
| [startOfMonth](control-flow-expression-language-functions.md#startOfMonth) | Devuelve el inicio del mes de una marca de tiempo. |
| [subtractFromTime](control-flow-expression-language-functions.md#subtractFromTime) | Resta un número de unidades de tiempo de una marca de tiempo. Consulte también [getPastTime](control-flow-expression-language-functions.md#getPastTime). |
| [ticks](control-flow-expression-language-functions.md#ticks) | Devuelve el valor de la propiedad `ticks` de una marca de tiempo especificada. |
| [utcNow](control-flow-expression-language-functions.md#utcNow) | Devuelve la marca de tiempo actual como una cadena. |

## <a name="function-reference"></a>Referencia de funciones

En esta sección se enumeran todas las funciones disponibles en orden alfabético.

<a name="add"></a>

### <a name="add"></a>add

Devuelve el resultado de sumar dos números.

```
add(<summand_1>, <summand_2>)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*sumando_1*>, <*sumando_2*> | Sí | Integer, Float o mixto | Números que se van a sumar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | -----| ----------- |
| <*resultado-de-la-suma*> | Integer o Float | Resultado de sumar los números especificados |
||||

*Ejemplo*

Este ejemplo suma los números especificados:

```
add(1, 1.5)
```

Y devuelve este resultado: `2.5`

<a name="addDays"></a>

### <a name="adddays"></a>addDays

Agrega un número de días a una marca de tiempo.

```
addDays('<timestamp>', <days>, '<format>'?)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*días*> | Sí | Entero | Número positivo o negativo de días que desea agregar |
| <*formato*> | No | String | Puede ser un [especificador de formato sencillo](/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo más el número de días especificado  |
||||

*Ejemplo 1*

Este ejemplo agrega 10 días a la marca de tiempo especificada:

```
addDays('2018-03-15T13:00:00Z', 10)
```

Y devuelve este resultado: `"2018-03-25T00:00:0000000Z"`

*Ejemplo 2*

Este ejemplo resta cinco días a la marca de tiempo especificada:

```
addDays('2018-03-15T00:00:00Z', -5)
```

Y devuelve este resultado: `"2018-03-10T00:00:0000000Z"`

<a name="addHours"></a>

### <a name="addhours"></a>addHours

Agrega un número de horas a una marca de tiempo.

```
addHours('<timestamp>', <hours>, '<format>'?)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*horas*> | Sí | Entero | Número positivo o negativo de horas que desea agregar |
| <*formato*> | No | String | Puede ser un [especificador de formato sencillo](/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo más el número de horas especificado  |
||||

*Ejemplo 1*

Este ejemplo agrega 10 horas a la marca de tiempo especificada:

```
addHours('2018-03-15T00:00:00Z', 10)
```

Y devuelve este resultado: `"2018-03-15T10:00:0000000Z"`

*Ejemplo 2*

Este ejemplo resta cinco horas a la marca de tiempo especificada:

```
addHours('2018-03-15T15:00:00Z', -5)
```

Y devuelve este resultado: `"2018-03-15T10:00:0000000Z"`

<a name="addMinutes"></a>

### <a name="addminutes"></a>addMinutes

Agrega un número de minutos a una marca de tiempo.

```
addMinutes('<timestamp>', <minutes>, '<format>'?)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*minutos*> | Sí | Entero | Número positivo o negativo de minutos que desea agregar |
| <*formato*> | No | String | Puede ser un [especificador de formato sencillo](/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo más el número de minutos especificado |
||||

*Ejemplo 1*

Este ejemplo agrega 10 minutos a la marca de tiempo especificada:

```
addMinutes('2018-03-15T00:10:00Z', 10)
```

Y devuelve este resultado: `"2018-03-15T00:20:00.0000000Z"`

*Ejemplo 2*

Este ejemplo resta cinco minutos a la marca de tiempo especificada:

```
addMinutes('2018-03-15T00:20:00Z', -5)
```

Y devuelve este resultado: `"2018-03-15T00:15:00.0000000Z"`

<a name="addSeconds"></a>

### <a name="addseconds"></a>addSeconds

Agrega un número de segundos a una marca de tiempo.

```
addSeconds('<timestamp>', <seconds>, '<format>'?)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*segundos*> | Sí | Entero | Número positivo o negativo de segundos que desea agregar |
| <*formato*> | No | String | Puede ser un [especificador de formato sencillo](/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo más el número de segundos especificado  |
||||

*Ejemplo 1*

Este ejemplo agrega 10 segundos a la marca de tiempo especificada:

```
addSeconds('2018-03-15T00:00:00Z', 10)
```

Y devuelve este resultado: `"2018-03-15T00:00:10.0000000Z"`

*Ejemplo 2*

Este ejemplo resta cinco segundos a la marca de tiempo especificada:

```
addSeconds('2018-03-15T00:00:30Z', -5)
```

Y devuelve este resultado: `"2018-03-15T00:00:25.0000000Z"`

<a name="addToTime"></a>

### <a name="addtotime"></a>addToTime

Agrega un número de unidades de tiempo a una marca de tiempo.
Consulte también [getFutureTime()](#getFutureTime).

```
addToTime('<timestamp>', <interval>, '<timeUnit>', '<format>'?)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*intervalo*> | Sí | Entero | Número de unidades de tiempo especificadas que se va a agregar |
| <*unidad_de_tiempo*> | Sí | String | La unidad de tiempo que se usará con *intervalo*: "Segundo", "Minuto", "Hora", "Día", "Semana", "Mes", "Año" |
| <*formato*> | No | String | Puede ser un [especificador de formato sencillo](/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo más el número de unidades de tiempo especificado  |
||||

*Ejemplo 1*

Este ejemplo agrega un día a la marca de tiempo especificada:

```
addToTime('2018-01-01T00:00:00Z', 1, 'Day')
```

Y devuelve este resultado: `"2018-01-02T00:00:00.0000000Z"`

*Ejemplo 2*

Este ejemplo agrega un día a la marca de tiempo especificada:

```
addToTime('2018-01-01T00:00:00Z', 1, 'Day', 'D')
```

Y devuelve el resultado con el formato "D" opcional: `"Tuesday, January 2, 2018"`

<a name="and"></a>

### <a name="and"></a>y

Compruebe si ambas expresiones son true.
Devuelve true si ambas expresiones son verdaderas, o bien devuelve false cuando al menos una expresión es falsa.

```
and(<expression1>, <expression2>)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*expresión1*>, <*expresión2*> | Sí | Boolean | Expresiones que se van a comprobar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | -----| ----------- |
| true o false | Boolean | Devuelve true cuando ambas expresiones son verdaderas. Devuelve false cuando al menos una expresión es falsa. |
||||

*Ejemplo 1*

Estos ejemplos comprueban si los dos valores booleanos especificados son verdaderos:

```
and(true, true)
and(false, true)
and(false, false)
```

Y devuelve estos resultados:

* Primer ejemplo: ambas expresiones son verdaderas, por lo que devuelve `true`.
* Segundo ejemplo: una expresión es falsa, por lo que devuelve `false`.
* Tercer ejemplo: ambas expresiones son falsas, por lo que devuelve `false`.

*Ejemplo 2*

Estos ejemplos comprueban si las dos expresiones especificadas son verdaderas:

```
and(equals(1, 1), equals(2, 2))
and(equals(1, 1), equals(1, 2))
and(equals(1, 2), equals(1, 3))
```

Y devuelve estos resultados:

* Primer ejemplo: ambas expresiones son verdaderas, por lo que devuelve `true`.
* Segundo ejemplo: una expresión es falsa, por lo que devuelve `false`.
* Tercer ejemplo: ambas expresiones son falsas, por lo que devuelve `false`.

<a name="array"></a>

### <a name="array"></a>array

Devuelve una matriz a partir de una única entrada especificada.
Para varias entradas, consulte [createArray()](#createArray).

```
array('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Cadena para la creación de una matriz |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| [<*valor*>] | Array | Matriz que contiene la única entrada especificada |
||||

*Ejemplo*

Este ejemplo crea una matriz a partir de la cadena "hello":

```
array('hello')
```

Y devuelve este resultado: `["hello"]`

<a name="base64"></a>

### <a name="base64"></a>base64

Devuelve la versión de una cadena codificada en base64.

```
base64('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Cadena de entrada |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*cadena-en-base64*> | String | Versión codificada en base64 de la cadena de entrada |
||||

*Ejemplo*

Este ejemplo convierte la cadena "hello" en una cadena codificada en base64:

```
base64('hello')
```

Y devuelve este resultado: `"aGVsbG8="`

<a name="base64ToBinary"></a>

### <a name="base64tobinary"></a>base64ToBinary

Devuelve la versión binaria de una cadena codificada en base64.

```
base64ToBinary('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Cadena con codificación base64 que se va a convertir |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*binario-de-cadena-en-base64*> | String | Versión binaria de la cadena con codificación base64 |
||||

*Ejemplo*

Este ejemplo convierte la cadena "aGVsbG8 =" codificada en base64 en una cadena binaria:

```
base64ToBinary('aGVsbG8=')
```

Y devuelve este resultado:

`"0110000101000111010101100111001101100010010001110011100000111101"`

<a name="base64ToString"></a>

### <a name="base64tostring"></a>base64ToString

Devuelve la versión de cadena de una cadena codificada en base64, decodificando dicha cadena en base64.
Use esta función en lugar de [decodeBase64()](#decodeBase64).
Aunque ambas funciones funcionan del mismo modo, `base64ToString()` es preferible.

```
base64ToString('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Cadena con codificación base64 que se va a decodificar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*cadena-base64-decodificada*> | String | Versión de cadena de una cadena codificada en base64 |
||||

*Ejemplo*

Este ejemplo convierte la cadena "aGVsbG8 =" codificada en base64 en una cadena:

```
base64ToString('aGVsbG8=')
```

Y devuelve este resultado: `"hello"`

<a name="binary"></a>

### <a name="binary"></a>binary

Devuelve la versión binaria de una cadena.

```
binary('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Cadena que se va a convertir |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*binario-del-valor-de-entrada*> | String | Versión binaria de la cadena especificada |
||||

*Ejemplo*

Este ejemplo convierte la cadena "hello" en una cadena en binario:

```
binary('hello')
```

Y devuelve este resultado:

`"0110100001100101011011000110110001101111"`

<a name="bool"></a>

### <a name="bool"></a>bool

Devuelve la versión booleana de un valor.

```
bool(<value>)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | Any | Valor que se va a convertir |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| true o false | Boolean | Versión booleana del valor especificado |
||||

*Ejemplo*

Estos ejemplos convierten los valores especificados en valores booleanos:

```
bool(1)
bool(0)
```

Y devuelve estos resultados:

* Primer ejemplo: `true`
* Segundo ejemplo: `false`

<a name="coalesce"></a>

### <a name="coalesce"></a>coalesce

Devuelve el primer valor distinto de null de uno o más parámetros.
Las cadenas vacías, las matrices vacías y los objetos vacíos no son nulos.

```
coalesce(<object_1>, <object_2>, ...)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*objeto_1*>, <*objeto_2*>, ... | Sí | Cualquiera, se pueden mezclar tipos | Uno o más elementos para comprobar si hay valores NULL |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*primer-elemento-no-NULL*> | Any | Primer elemento o valor que no sea NULL. Si todos los parámetros son NULL, esta función devuelve NULL. |
||||

*Ejemplo*

Estos ejemplos devuelven el primer valor distinto de NULL de los valores especificados o NULL cuando todos los valores son NULL:

```
coalesce(null, true, false)
coalesce(null, 'hello', 'world')
coalesce(null, null, null)
```

Y devuelve estos resultados:

* Primer ejemplo: `true`
* Segundo ejemplo: `"hello"`
* Tercer ejemplo: `null`

<a name="concat"></a>

### <a name="concat"></a>concat

Combina dos o más cadenas y devuelve la cadena combinada.

```
concat('<text1>', '<text2>', ...)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*texto1*>, <*texto2*>, ... | Sí | String | Al menos dos cadenas para combinar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*texto1texto2...* > | String | Cadena que se crea a partir de las cadenas de entrada combinadas |
||||

*Ejemplo*

Este ejemplo combina las cadenas "Hello" y "World":

```
concat('Hello', 'World')
```

Y devuelve este resultado: `"HelloWorld"`

<a name="contains"></a>

### <a name="contains"></a>contains

Comprueba si una colección contiene un elemento específico.
Devuelve true cuando se encuentra el elemento o devuelve false cuando no se encuentra.
Esta función distingue mayúsculas de minúsculas.

```
contains('<collection>', '<value>')
contains([<collection>], '<value>')
```

En concreto, esta función funciona en estos tipos de colección:

* Una *cadena* para buscar una *subcadena*
* Una *matriz* para buscar un *valor*
* Un *diccionario* para buscar una *clave*

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*colección*> | Sí | Cadena, matriz o diccionario | Colección que se va a comprobar |
| <*value*> | Sí | Cadena, matriz o diccionario, respectivamente | Elemento que se va a buscar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| true o false | Boolean | Devuelve true cuando se encuentra el elemento. Devuelve false si no se encuentra. |
||||

*Ejemplo 1*

Este ejemplo comprueba la cadena "hello world" para buscar la subcadena "world" y devuelve true:

```
contains('hello world', 'world')
```

*Ejemplo 2*

Este ejemplo comprueba la cadena "hello world" para buscar la subcadena "universe" y devuelve false:

```
contains('hello world', 'universe')
```

<a name="convertFromUtc"></a>

### <a name="convertfromutc"></a>convertFromUtc

Convierte una marca de tiempo del formato Hora Universal Coordinada (UTC) a la zona horaria de destino.

```
convertFromUtc('<timestamp>', '<destinationTimeZone>', '<format>'?)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*zona_horaria_de_destino*> | Sí | String | Nombre de la zona horaria de destino. Para los nombres de zonas horarias, consulte [Valores de índice de zona horaria de Microsoft](https://support.microsoft.com/help/973627/microsoft-time-zone-index-values), pero es posible que tenga que quitar los signos de puntuación del nombre de zona horaria. |
| <*formato*> | No | String | Puede ser un [especificador de formato sencillo](/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-convertida*> | String | Marca de tiempo convertida a la zona horaria de destino |
||||

*Ejemplo 1*

Este ejemplo convierte una marca de tiempo a la zona horaria especificada:

```
convertFromUtc('2018-01-01T08:00:00.0000000Z', 'Pacific Standard Time')
```

Y devuelve este resultado: `"2018-01-01T00:00:00Z"`

*Ejemplo 2*

Este ejemplo convierte una marca de tiempo a la zona horaria y el formato especificados:

```
convertFromUtc('2018-01-01T08:00:00.0000000Z', 'Pacific Standard Time', 'D')
```

Y devuelve este resultado: `"Monday, January 1, 2018"`

<a name="convertTimeZone"></a>

### <a name="converttimezone"></a>convertTimeZone

Convierte una marca de tiempo de la zona horaria de origen a la zona horaria de destino.

```
convertTimeZone('<timestamp>', '<sourceTimeZone>', '<destinationTimeZone>', '<format>'?)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*zona_horaria_de_origen*> | Sí | String | Nombre de la zona horaria de origen. Para los nombres de zonas horarias, consulte [Valores de índice de zona horaria de Microsoft](https://support.microsoft.com/help/973627/microsoft-time-zone-index-values), pero es posible que tenga que quitar los signos de puntuación del nombre de zona horaria. |
| <*zona_horaria_de_destino*> | Sí | String | Nombre de la zona horaria de destino. Para los nombres de zonas horarias, consulte [Valores de índice de zona horaria de Microsoft](https://support.microsoft.com/help/973627/microsoft-time-zone-index-values), pero es posible que tenga que quitar los signos de puntuación del nombre de zona horaria. |
| <*formato*> | No | String | Puede ser un [especificador de formato sencillo](/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-convertida*> | String | Marca de tiempo convertida a la zona horaria de destino |
||||

*Ejemplo 1*

Este ejemplo convierte la zona horaria de origen a la zona horaria de destino:

```
convertTimeZone('2018-01-01T08:00:00.0000000Z', 'UTC', 'Pacific Standard Time')
```

Y devuelve este resultado: `"2018-01-01T00:00:00.0000000"`

*Ejemplo 2*

Este ejemplo convierte una zona horaria a la zona horaria y el formato especificados:

```
convertTimeZone('2018-01-01T08:00:00.0000000Z', 'UTC', 'Pacific Standard Time', 'D')
```

Y devuelve este resultado: `"Monday, January 1, 2018"`

<a name="convertToUtc"></a>

### <a name="converttoutc"></a>convertToUtc

Convierte una marca de tiempo de la zona horaria de origen al formato Hora Universal Coordinada (UTC).

```
convertToUtc('<timestamp>', '<sourceTimeZone>', '<format>'?)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*zona_horaria_de_origen*> | Sí | String | Nombre de la zona horaria de origen. Para los nombres de zonas horarias, consulte [Valores de índice de zona horaria de Microsoft](https://support.microsoft.com/help/973627/microsoft-time-zone-index-values), pero es posible que tenga que quitar los signos de puntuación del nombre de zona horaria. |
| <*formato*> | No | String | Puede ser un [especificador de formato sencillo](/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-convertida*> | String | Marca de tiempo convertida a formato UTC |
||||

*Ejemplo 1*

Este ejemplo convierte una marca de tiempo a formato UTC:

```
convertToUtc('01/01/2018 00:00:00', 'Pacific Standard Time')
```

Y devuelve este resultado: `"2018-01-01T08:00:00.0000000Z"`

*Ejemplo 2*

Este ejemplo convierte una marca de tiempo a formato UTC:

```
convertToUtc('01/01/2018 00:00:00', 'Pacific Standard Time', 'D')
```

Y devuelve este resultado: `"Monday, January 1, 2018"`

<a name="createArray"></a>

### <a name="createarray"></a>createArray

Devuelve una matriz a partir de varias entradas.
Para matrices de una sola entrada, consulte [array()](#array).

```
createArray('<object1>', '<object2>', ...)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*objeto1*>, <*objeto2*>,... | Sí | Cualquiera, pero no mixtos | Al menos dos elementos para crear la matriz |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| [<*objeto1*>, <*objeto2*>, ...] | Array | Matriz creada a partir de todos los elementos de entrada |
||||

*Ejemplo*

Este ejemplo crea una matriz a partir de estas entradas:

```
createArray('h', 'e', 'l', 'l', 'o')
```

Y devuelve este resultado: `["h", "e", "l", "l", "o"]`

<a name="dataUri"></a>

### <a name="datauri"></a>dataUri

Devuelve un identificador uniforme de recursos (URI) de datos de una cadena.

```
dataUri('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Cadena que se va a convertir |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*uri-de-datos*> | String | Identificador URI de datos de la cadena de entrada |
||||

*Ejemplo*

Este ejemplo crea un identificador URI de datos a partir de la cadena "hello":

```
dataUri('hello')
```

Y devuelve este resultado: `"data:text/plain;charset=utf-8;base64,aGVsbG8="`

<a name="dataUriToBinary"></a>

### <a name="datauritobinary"></a>dataUriToBinary

Devuelve la versión binaria de un identificador uniforme de recursos (URI) de datos.
Use esta función en lugar de [decodeDataUri()](#decodeDataUri).
Aunque ambas funciones funcionan del mismo modo, `dataUriBinary()` es preferible.

```
dataUriToBinary('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Identificador URI de datos que se va a convertir |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*binario-de-uri-de-datos*> | String | Versión binaria del identificador URI de datos |
||||

*Ejemplo*

Este ejemplo crea una versión binaria de este identificador URI de datos:

```
dataUriToBinary('data:text/plain;charset=utf-8;base64,aGVsbG8=')
```

Y devuelve este resultado:

`"01100100011000010111010001100001001110100111010001100101011110000111010000101111011100000
1101100011000010110100101101110001110110110001101101000011000010111001001110011011001010111
0100001111010111010101110100011001100010110100111000001110110110001001100001011100110110010
10011011000110100001011000110000101000111010101100111001101100010010001110011100000111101"`

<a name="dataUriToString"></a>

### <a name="datauritostring"></a>dataUriToString

Devuelve la versión de cadena de un identificador uniforme de recursos (URI) de datos.

```
dataUriToString('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Identificador URI de datos que se va a convertir |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*cadena-de-uri-de-datos*> | String | Versión de cadena del identificador URI de datos |
||||

*Ejemplo*

Este ejemplo crea una cadena a partir de este identificador URI de datos:

```
dataUriToString('data:text/plain;charset=utf-8;base64,aGVsbG8=')
```

Y devuelve este resultado: `"hello"`

<a name="dayOfMonth"></a>

### <a name="dayofmonth"></a>dayOfMonth

Devuelve el día del mes de una marca de tiempo.

```
dayOfMonth('<timestamp>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*día-del-mes*> | Entero | Día del mes de la marca de tiempo especificada |
||||

*Ejemplo*

Este ejemplo devuelve el número del día del mes de esta marca de tiempo:

```
dayOfMonth('2018-03-15T13:27:36Z')
```

Y devuelve este resultado: `15`

<a name="dayOfWeek"></a>

### <a name="dayofweek"></a>dayOfWeek

Devuelve el día de la semana de una marca de tiempo.

```
dayOfWeek('<timestamp>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*día-de-la-semana*> | Entero | Día de la semana de la marca de tiempo especificada, donde el domingo es 0, el lunes es 1, etc. |
||||

*Ejemplo*

Este ejemplo devuelve el número del día de la semana de esta marca de tiempo:

```
dayOfWeek('2018-03-15T13:27:36Z')
```

Y devuelve este resultado: `3`

<a name="dayOfYear"></a>

### <a name="dayofyear"></a>dayOfYear

Devuelve el día del año de una marca de tiempo.

```
dayOfYear('<timestamp>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*día-del-año*> | Entero | Día del año de la marca de tiempo especificada |
||||

*Ejemplo*

Este ejemplo devuelve el número del día del año de esta marca de tiempo:

```
dayOfYear('2018-03-15T13:27:36Z')
```

Y devuelve este resultado: `74`

<a name="decodeBase64"></a>

### <a name="decodebase64"></a>decodeBase64

Devuelve la versión de cadena de una cadena codificada en base64, decodificando dicha cadena en base64.
Considere el uso de [base64ToString()](#base64ToString) en lugar de `decodeBase64()`.
Aunque ambas funciones funcionan del mismo modo, `base64ToString()` es preferible.

```
decodeBase64('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Cadena con codificación base64 que se va a decodificar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*cadena-base64-decodificada*> | String | Versión de cadena de una cadena codificada en base64 |
||||

*Ejemplo*

Este ejemplo crea una cadena a partir de una cadena codificada en base64:

```
decodeBase64('aGVsbG8=')
```

Y devuelve este resultado: `"hello"`

<a name="decodeDataUri"></a>

### <a name="decodedatauri"></a>decodeDataUri

Devuelve la versión binaria de un identificador uniforme de recursos (URI) de datos.
Considere el uso de [dataUriToBinary()](#dataUriToBinary) en lugar de `decodeDataUri()`.
Aunque ambas funciones funcionan del mismo modo, `dataUriToBinary()` es preferible.

```
decodeDataUri('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Cadena URI de datos que se va a decodificar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*binario-de-uri-de-datos*> | String | Versión binaria de una cadena URI de datos |
||||

*Ejemplo*

Este ejemplo devuelve la versión binaria de este identificador URI de datos:

```
decodeDataUri('data:text/plain;charset=utf-8;base64,aGVsbG8=')
```

Y devuelve este resultado:

`"01100100011000010111010001100001001110100111010001100101011110000111010000101111011100000
1101100011000010110100101101110001110110110001101101000011000010111001001110011011001010111
0100001111010111010101110100011001100010110100111000001110110110001001100001011100110110010
10011011000110100001011000110000101000111010101100111001101100010010001110011100000111101"`

<a name="decodeUriComponent"></a>

### <a name="decodeuricomponent"></a>decodeUriComponent

Devuelve una cadena que reemplaza los caracteres de escape por versiones descodificadas.

```
decodeUriComponent('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Cadena con caracteres de escape que se va a decodificar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*uri-decodificado*> | String | Cadena actualizada con los caracteres de escape decodificados |
||||

*Ejemplo*

Este ejemplo reemplaza los caracteres de escape de esta cadena por versiones decodificadas:

```
decodeUriComponent('http%3A%2F%2Fcontoso.com')
```

Y devuelve este resultado: `"https://contoso.com"`

<a name="div"></a>

### <a name="div"></a>div

Devuelve el resultado entero de dividir dos números.
Para obtener el resultado del resto, consulte [mod()](#mod).

```
div(<dividend>, <divisor>)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*dividend*> | Sí | Integer o Float | Número que se va a dividir entre el *divisor*. |
| <*divisor*> | Sí | Integer o Float | Número que divide el *dividendo*, pero no puede ser 0 |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*resultado-cociente*> | Entero | Resultado entero de dividir el primer número entre el segundo número |
||||

*Ejemplo*

Ambos ejemplos dividen el primer número entre el segundo número:

```
div(10, 5)
div(11, 5)
```

Y devuelven este resultado: `2`

<a name="encodeUriComponent"></a>

### <a name="encodeuricomponent"></a>encodeUriComponent

Devuelve una versión codificada de un identificador uniforme de recursos (URI) de una cadena reemplazando los caracteres no seguros de la dirección URL por caracteres de escape.
Considere el uso de [uriComponent()](#uriComponent) en lugar de `encodeUriComponent()`.
Aunque ambas funciones funcionan del mismo modo, `uriComponent()` es preferible.

```
encodeUriComponent('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Cadena que se va a convertir en formato codificado de URI |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*uri-codificado*> | String | Cadena codificada en formato URI con caracteres de escape |
||||

*Ejemplo*

Este ejemplo crea una versión codificada en formato URI de esta cadena:

```
encodeUriComponent('https://contoso.com')
```

Y devuelve este resultado: `"http%3A%2F%2Fcontoso.com"`

<a name="empty"></a>

### <a name="empty"></a>empty

Comprueba si una colección está vacía.
Devuelve true cuando la colección está vacía o devuelve false cuando no está vacía.

```
empty('<collection>')
empty([<collection>])
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*colección*> | Sí | Cadena, matriz u objeto | Colección que se va a comprobar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| true o false | Boolean | Devuelve true cuando la colección está vacía. Devuelve false si no está vacía. |
||||

*Ejemplo*

Estos ejemplos comprueban si las colecciones especificadas están vacías:

```
empty('')
empty('abc')
```

Y devuelve estos resultados:

* Primer ejemplo: pasa una cadena vacía, por lo que la función devuelve `true`.
* Segundo ejemplo: pasa la cadena "abc", por lo que la función devuelve `false`.

<a name="endswith"></a>

### <a name="endswith"></a>endsWith

Comprueba si una cadena termina con una subcadena especificada.
Devuelve true cuando se encuentra la subcadena o devuelve false cuando no se encuentra.
Esta función no distingue mayúsculas de minúsculas.

```
endsWith('<text>', '<searchText>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena que se va a comprobar |
| <*texto_a_buscar*> | Sí | String | Subcadena final que se va a buscar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| true o false  | Boolean | Devuelve true cuando se encuentra la subcadena final. Devuelve false si no se encuentra. |
||||

*Ejemplo 1*

Este ejemplo comprueba si la cadena "hello world" termina con la cadena "world":

```
endsWith('hello world', 'world')
```

Y devuelve este resultado: `true`

*Ejemplo 2*

Este ejemplo comprueba si la cadena "hello world" termina con la cadena "universe":

```
endsWith('hello world', 'universe')
```

Y devuelve este resultado: `false`

<a name="equals"></a>

### <a name="equals"></a>equals

Comprueba si los valores, expresiones u objetos son equivalentes.
Devuelve true cuando ambos son equivalentes o devuelve false cuando no son equivalentes.

```
equals('<object1>', '<object2>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*objeto1*>, <*objeto2*> | Sí | Varios | Valores, expresiones u objetos que se van a comparar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| true o false | Boolean | Devuelve true cuando ambos son equivalentes. Devuelve false cuando no son equivalentes. |
||||

*Ejemplo*

Estos ejemplos comprueban si las entradas especificadas son equivalentes.

```
equals(true, 1)
equals('abc', 'abcd')
```

Y devuelve estos resultados:

* Primer ejemplo: ambos valores son equivalentes, por lo que la función devuelve `true`.
* Segundo ejemplo: ambos valores no son equivalentes, por lo que la función devuelve `false`.

<a name="first"></a>

### <a name="first"></a>first

Devuelve el primer elemento de una cadena o una matriz.

```
first('<collection>')
first([<collection>])
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*colección*> | Sí | Cadena o matriz | Colección en la que buscar el primer elemento |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*primer-elemento-de-la-colección*> | Any | Primer elemento de la colección |
||||

*Ejemplo*

Estos ejemplos buscan el primer elemento de estas colecciones:

```
first('hello')
first(createArray(0, 1, 2))
```

Asimismo, devuelven estos resultados:

* Primer ejemplo: `"h"`
* Segundo ejemplo: `0`

<a name="float"></a>

### <a name="float"></a>FLOAT

Convierte una versión de cadena de un número de punto flotante a un número de punto flotante real.

```
float('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Cadena que contiene un número de punto flotante válido que se va a convertir |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*valor-de-tipo-float*> | Float | Número de punto flotante a partir de la cadena especificada |
||||

*Ejemplo*

Este ejemplo crea una versión de cadena para este número de punto flotante:

```
float('10.333')
```

Y devuelve este resultado: `10.333`

<a name="formatDateTime"></a>

### <a name="formatdatetime"></a>formatDateTime

Devuelve una marca de tiempo en el formato especificado.

```
formatDateTime('<timestamp>', '<format>'?)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*formato*> | No | String | Puede ser un [especificador de formato sencillo](/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-con-nuevo-formato*> | String | Marca de tiempo actualizada en el formato especificado |
||||

*Ejemplo*

Este ejemplo convierte una marca de tiempo al formato especificado:

```
formatDateTime('03/15/2018 12:00:00', 'yyyy-MM-ddTHH:mm:ss')
```

Y devuelve este resultado: `"2018-03-15T12:00:00"`

<a name="getFutureTime"></a>

### <a name="getfuturetime"></a>getFutureTime

Devuelve la marca de tiempo actual más las unidades de tiempo especificadas.

```
getFutureTime(<interval>, <timeUnit>, <format>?)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*intervalo*> | Sí | Entero | Número de unidades de tiempo especificadas que se va a agregar |
| <*unidad_de_tiempo*> | Sí | String | La unidad de tiempo que se usará con *intervalo*: "Segundo", "Minuto", "Hora", "Día", "Semana", "Mes", "Año" |
| <*formato*> | No | String | Puede ser un [especificador de formato sencillo](/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo actual más el número de unidades de tiempo especificado |
||||

*Ejemplo 1*

Imagine que la marca de tiempo actual es "2018-03-01T00:00:00.0000000Z".
Este ejemplo agrega cinco días a esa marca de tiempo:

```
getFutureTime(5, 'Day')
```

Y devuelve este resultado: `"2018-03-06T00:00:00.0000000Z"`

*Ejemplo 2*

Imagine que la marca de tiempo actual es "2018-03-01T00:00:00.0000000Z".
Este ejemplo agrega cinco días y convierte el resultado al formato "D":

```
getFutureTime(5, 'Day', 'D')
```

Y devuelve este resultado: `"Tuesday, March 6, 2018"`

<a name="getPastTime"></a>

### <a name="getpasttime"></a>getPastTime

Devuelve la marca de tiempo actual menos las unidades de tiempo especificadas.

```
getPastTime(<interval>, <timeUnit>, <format>?)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*intervalo*> | Sí | Entero | Número de unidades de tiempo especificadas que se va a sustraer |
| <*unidad_de_tiempo*> | Sí | String | La unidad de tiempo que se usará con *intervalo*: "Segundo", "Minuto", "Hora", "Día", "Semana", "Mes", "Año" |
| <*formato*> | No | String | Puede ser un [especificador de formato sencillo](/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo actual menos el número de unidades de tiempo especificado |
||||

*Ejemplo 1*

Imagine que la marca de tiempo actual es "2018-02-01T00:00:00.0000000Z".
Este ejemplo resta cinco días a esa marca de tiempo:

```
getPastTime(5, 'Day')
```

Y devuelve este resultado: `"2018-01-27T00:00:00.0000000Z"`

*Ejemplo 2*

Imagine que la marca de tiempo actual es "2018-02-01T00:00:00.0000000Z".
Este ejemplo resta cinco días y convierte el resultado al formato "D":

```
getPastTime(5, 'Day', 'D')
```

Y devuelve este resultado: `"Saturday, January 27, 2018"`

<a name="greater"></a>

### <a name="greater"></a>greater

Comprueba si el primer valor es mayor que el segundo.
Devuelve true cuando el primer valor es mayor o devuelve false cuando es menor.

```
greater(<value>, <compareTo>)
greater('<value>', '<compareTo>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | Integer, Float o String | Primer valor para comprobar si es mayor que el segundo valor |
| <*compareTo*> | Sí | Integer, Float o String, respectivamente | Valor de comparación |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| true o false | Boolean | Devuelve true si el primer valor es mayor que el segundo. Devuelve false si el primer valor es igual o menor que el segundo. |
||||

*Ejemplo*

Estos ejemplos comprueban si el primer valor es mayor que el segundo:

```
greater(10, 5)
greater('apple', 'banana')
```

Asimismo, devuelven estos resultados:

* Primer ejemplo: `true`
* Segundo ejemplo: `false`

<a name="greaterOrEquals"></a>

### <a name="greaterorequals"></a>greaterOrEquals

Comprueba si el primer valor es mayor o igual que el segundo.
Devuelve true cuando el primer valor es mayor o igual o devuelve false cuando el primer valor es menor.

```
greaterOrEquals(<value>, <compareTo>)
greaterOrEquals('<value>', '<compareTo>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | Integer, Float o String | Primer valor para comprobar si es mayor o igual que el segundo valor |
| <*compareTo*> | Sí | Integer, Float o String, respectivamente | Valor de comparación |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| true o false | Boolean | Devuelve true si el primer valor es mayor o igual que el segundo. Devuelve false si el primer valor es menor que el segundo. |
||||

*Ejemplo*

Estos ejemplos comprueban si el primer valor es mayor o igual que el segundo:

```
greaterOrEquals(5, 5)
greaterOrEquals('apple', 'banana')
```

Asimismo, devuelven estos resultados:

* Primer ejemplo: `true`
* Segundo ejemplo: `false`

<a name="guid"></a>

### <a name="guid"></a>guid

Genera un identificador único global (GUID) como una cadena, por ejemplo, "c2ecc88d-88c8-4096-912c-d6f2e2b138ce":

```
guid()
```

Además, es posible especificar un formato diferente del GUID que no sea el formato predeterminado, "D", que es 32 dígitos separados por guiones.

```
guid('<format>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*formato*> | No | String | Un único [especificador de formato](/dotnet/api/system.guid.tostring) para el GUID devuelto. De forma predeterminada, el formato es "D", pero puede usar "N", "D", "B", "P" o "X". |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*valor-de-GUID*> | String | GUID generado aleatoriamente |
||||

*Ejemplo*

Este ejemplo genera el mismo GUID, pero como 32 dígitos separados por guiones y entre paréntesis:

```
guid('P')
```

Y devuelve este resultado: `"(c2ecc88d-88c8-4096-912c-d6f2e2b138ce)"`

<a name="if"></a>

### <a name="if"></a>if

Comprueba si una expresión es true o false.
En función del resultado, devuelve un valor especificado.

```
if(<expression>, <valueIfTrue>, <valueIfFalse>)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*expression*> | Sí | Boolean | Expresión que se va a evaluar |
| <*valor_si_es_true*> | Sí | Any | Valor que se devuelve cuando la expresión es verdadera |
| <*valor_si_es_false*> | Sí | Any | Valor que se devuelve cuando la expresión es falsa |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*valor-a-devolver-especificado*> | Any | Valor especificado que se devuelve en función de si la expresión es true o false |
||||

*Ejemplo*

Este ejemplo devuelve `"yes"` porque la expresión especificada devuelve true.
En caso contrario, el ejemplo devuelve `"no"`:

```
if(equals(1, 1), 'yes', 'no')
```

<a name="indexof"></a>

### <a name="indexof"></a>indexOf

Devuelve la posición de inicio o valor de índice de una subcadena.
Esta función no distingue entre mayúsculas y minúsculas y los índices comienzan por el número 0.

```
indexOf('<text>', '<searchText>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena que contiene la subcadena que se va a buscar |
| <*texto_a_buscar*> | Sí | String | Subcadena que se va a buscar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*valor-de-índice*>| Entero | Posición de inicio o valor de índice de la subcadena especificada. <p>Si no se encuentra la cadena, devuelve el número -1. |
||||

*Ejemplo*

Este ejemplo busca el valor de índice inicial de la subcadena "world" en la cadena "hello world":

```
indexOf('hello world', 'world')
```

Y devuelve este resultado: `6`

<a name="int"></a>

### <a name="int"></a>int

Devuelve la versión como número entero de una cadena.

```
int('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Cadena que se va a convertir |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*resultado-de-número-entero*> | Entero | Versión como número entero de la cadena especificada. |
||||

*Ejemplo*

Este ejemplo crea una versión como número entero de la cadena "10":

```
int('10')
```

Y devuelve este resultado: `10`

<a name="json"></a>

### <a name="json"></a>json

Devuelve el valor o el objeto de tipo Notación de objetos JavaScript (JSON) de una cadena o XML.

```
json('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String o XML | Cadena o XML que se va a convertir |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*resultado-JSON*> | Objeto o tipo nativo de JSON | Objeto o valor de tipo nativo de JSON de la cadena o XML especificados. Si la cadena es NULL, la función devuelve un objeto vacío. |
||||

*Ejemplo 1*

Este ejemplo convierte esta cadena en el valor JSON:

```
json('[1, 2, 3]')
```

Y devuelve este resultado: `[1, 2, 3]`

*Ejemplo 2*

Este ejemplo convierte esta cadena a JSON:

```
json('{"fullName": "Sophia Owen"}')
```

Y devuelve este resultado:

```
{
  "fullName": "Sophia Owen"
}
```

*Ejemplo 3*

Este ejemplo convierte este XML a JSON:

```
json(xml('<?xml version="1.0"?> <root> <person id='1'> <name>Sophia Owen</name> <occupation>Engineer</occupation> </person> </root>'))
```

Y devuelve este resultado:

```json
{
   "?xml": { "@version": "1.0" },
   "root": {
      "person": [ {
         "@id": "1",
         "name": "Sophia Owen",
         "occupation": "Engineer"
      } ]
   }
}
```

<a name="intersection"></a>

### <a name="intersection"></a>intersección

Devuelve una colección que tiene *solo* los elementos comunes en las colecciones especificadas.
Para que aparezca en el resultado, un elemento debe aparecer en todas las colecciones que se pasan a esta función.
Si uno o más elementos tienen el mismo nombre, el último elemento con ese nombre aparece en el resultado.

```
intersection([<collection1>], [<collection2>], ...)
intersection('<collection1>', '<collection2>', ...)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*colección1*>, <*colección2*>, ... | Sí | Matriz u objeto, pero no ambos | Colecciones de las que desea *solo* los elementos comunes |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*elementos-comunes*> | Matriz u objeto, respectivamente | Colección que tiene solo los elementos comunes en las colecciones especificadas |
||||

*Ejemplo*

Este ejemplo busca los elementos comunes en estas matrices:

```
intersection(createArray(1, 2, 3), createArray(101, 2, 1, 10), createArray(6, 8, 1, 2))
```

Y devuelve una matriz con *solo* estos elementos: `[1, 2]`

<a name="join"></a>

### <a name="join"></a>join

Devuelve una cadena que tiene todos los elementos de una matriz y tiene cada carácter separado por un *delimitador*.

```
join([<collection>], '<delimiter>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*colección*> | Sí | Array | Matriz que tiene los elementos que se van a unir |
| <*delimitador*> | Sí | String | Separador que aparece entre cada carácter de la cadena resultante |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*carácter1*><*delimitador*><*carácter2*><*delimitador*>... | String | Cadena resultante creada a partir de todos los elementos de la matriz especificada |
||||

*Ejemplo*

Este ejemplo crea una cadena a partir de todos los elementos de esta matriz con el carácter especificado como delimitador:

```
join(createArray('a', 'b', 'c'), '.')
```

Y devuelve este resultado: `"a.b.c"`

<a name="last"></a>

### <a name="last"></a>last

Devuelve el último elemento de una colección.

```
last('<collection>')
last([<collection>])
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*colección*> | Sí | Cadena o matriz | Colección en la que buscar el último elemento |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*último-elemento-de-la-colección*> | Cadena o matriz, respectivamente | Último elemento de la colección |
||||

*Ejemplo*

Estos ejemplos buscan el último elemento de estas colecciones:

```
last('abcd')
last(createArray(0, 1, 2, 3))
```

Y devuelve estos resultados:

* Primer ejemplo: `"d"`
* Segundo ejemplo: `3`

<a name="lastindexof"></a>

### <a name="lastindexof"></a>lastIndexOf

Devuelve la posición inicial o el valor de índice de la última repetición de una subcadena.
Esta función no distingue entre mayúsculas y minúsculas y los índices comienzan por el número 0.

```
lastIndexOf('<text>', '<searchText>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena que contiene la subcadena que se va a buscar |
| <*texto_a_buscar*> | Sí | String | Subcadena que se va a buscar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*valor-de-índice-final*> | Entero | Posición inicial o valor de índice de la última repetición de la subcadena especificada. <p>Si no se encuentra la cadena, devuelve el número -1. |
||||

*Ejemplo*

Este ejemplo busca el valor de índice inicial de la última repetición de la subcadena "world" en la cadena "hello world":

```
lastIndexOf('hello world', 'world')
```

Y devuelve este resultado: `6`

<a name="length"></a>

### <a name="length"></a>length

Devuelve el número de elementos de una colección.

```
length('<collection>')
length([<collection>])
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*colección*> | Sí | Cadena o matriz | Colección con los elementos que se van a contar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*length-or-count*> | Entero | Número de elementos de la colección |
||||

*Ejemplo*

Estos ejemplos cuentan el número de elementos de estas colecciones:

```
length('abcd')
length(createArray(0, 1, 2, 3))
```

Y devuelven este resultado: `4`

<a name="less"></a>

### <a name="less"></a>less

Comprueba si el primer valor es menor que el segundo.
Devuelve true cuando el primer valor es menor o devuelve false cuando el primer valor es mayor.

```
less(<value>, <compareTo>)
less('<value>', '<compareTo>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | Integer, Float o String | Primer valor para comprobar si es menor que el segundo valor |
| <*compareTo*> | Sí | Integer, Float o String, respectivamente | Elemento de comparación |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| true o false | Boolean | Devuelve true si el primer valor es menor que el segundo valor. Devuelve false si el primer valor es igual o mayor que el segundo. |
||||

*Ejemplo*

En estos ejemplos se comprueba si el primer valor es menor que el segundo valor.

```
less(5, 10)
less('banana', 'apple')
```

Asimismo, devuelven estos resultados:

* Primer ejemplo: `true`
* Segundo ejemplo: `false`

<a name="lessOrEquals"></a>

### <a name="lessorequals"></a>lessOrEquals

Compruebe si el primer valor es menor o igual que el segundo valor.
Devuelve "true" cuando el primer valor es menor o igual, o bien devuelve "false" si el primer valor es mayor.

```
lessOrEquals(<value>, <compareTo>)
lessOrEquals('<value>', '<compareTo>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | Integer, Float o String | Primer valor que se va a comprobar si es menor o igual que el segundo valor. |
| <*compareTo*> | Sí | Integer, Float o String, respectivamente | Elemento de comparación |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| true o false  | Boolean | Devuelve true si el primer valor es menor o igual que el segundo. Devuelve false si el primer valor es mayor que el segundo. |
||||

*Ejemplo*

Estos ejemplos comprueban si el primer valor es menor o igual que el segundo.

```
lessOrEquals(10, 10)
lessOrEquals('apply', 'apple')
```

Asimismo, devuelven estos resultados:

* Primer ejemplo: `true`
* Segundo ejemplo: `false`

<a name="max"></a>

### <a name="max"></a>max

Devuelve el valor más alto de una lista o matriz con números, incluyendo ambos extremos.

```
max(<number1>, <number2>, ...)
max([<number1>, <number2>, ...])
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*number1*>, <*number2*>, ... | Sí | Integer, Float o ambos | Conjunto de números del que se desea obtener el valor más alto |
| [<*número1*>, <*número2*>,...] | Sí | Matriz: Integer, Float o ambos | Matriz de números de la cual quiere obtener el valor más alto |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*max-value*> | Integer o Float | Valor más alto en la matriz o conjunto de números especificados |
||||

*Ejemplo*

Estos ejemplos obtienen el valor más alto del conjunto de números y la matriz:

```
max(1, 2, 3)
max(createArray(1, 2, 3))
```

Y devuelven este resultado: `3`

<a name="min"></a>

### <a name="min"></a>Min

Devuelve el valor más bajo de un conjunto de números o una matriz.

```
min(<number1>, <number2>, ...)
min([<number1>, <number2>, ...])
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*number1*>, <*number2*>, ... | Sí | Integer, Float o ambos | Conjunto de números del que se desea obtener el valor más bajo |
| [<*número1*>, <*número2*>,...] | Sí | Matriz: Integer, Float o ambos | Matriz de números de la que se desea obtener el valor más bajo |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*min-value*> | Integer o Float | Valor más bajo de la matriz o conjunto de números especificados |
||||

*Ejemplo*

Estos ejemplos obtienen el valor más bajo del conjunto de números y la matriz:

```
min(1, 2, 3)
min(createArray(1, 2, 3))
```

Y devuelven este resultado: `1`

<a name="mod"></a>

### <a name="mod"></a>mod

Devuelve el resto de dividir dos números.
Para obtener el resultado de número entero, consulte [div()](#div).

```
mod(<dividend>, <divisor>)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*dividend*> | Sí | Integer o Float | Número que se va a dividir entre el *divisor*. |
| <*divisor*> | Sí | Integer o Float | Número que divide el *dividendo*, pero no puede ser 0. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*modulo-result*> | Integer o Float | Resto de dividir el primer número entre el segundo número |
||||

*Ejemplo*

En este ejemplo se divide el primer número por el segundo número:

```
mod(3, 2)
```

Y devuelven este resultado: `1`

<a name="mul"></a>

### <a name="mul"></a>mul

Devuelve el producto de multiplicar dos números.

```
mul(<multiplicand1>, <multiplicand2>)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*multiplicand1*> | Sí | Integer o Float | Número que se va a multiplicar por el *multiplicando2* |
| <*multiplicand2*> | Sí | Integer o Float | Número que multiplica al *multiplicando1* |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*product-result*> | Integer o Float | Producto de multiplicar el primer número por el segundo número |
||||

*Ejemplo*

Estos ejemplos multiplican el primer número por el segundo número:

```
mul(1, 2)
mul(1.5, 2)
```

Asimismo, devuelven estos resultados:

* Primer ejemplo: `2`
* Segundo ejemplo: `3`

<a name="not"></a>

### <a name="not"></a>not

Comprueba si una expresión es false.
Devuelve verdadero cuando la expresión es falsa o devuelve false cuando es verdadera.

```json
not(<expression>)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*expression*> | Sí | Boolean | Expresión que se va a evaluar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| true o false | Boolean | Devuelve true cuando la expresión es falsa. Devuelve false cuando la expresión es verdadera. |
||||

*Ejemplo 1*

Estos ejemplos comprueban si las expresiones especificadas son falsas:

```json
not(false)
not(true)
```

Asimismo, devuelven estos resultados:

* Primer ejemplo: la expresión es falsa, por lo que la función devuelve `true`.
* Segundo ejemplo: la expresión es verdadera, por lo que la función devuelve `false`.

*Ejemplo 2*

Estos ejemplos comprueban si las expresiones especificadas son falsas:

```json
not(equals(1, 2))
not(equals(1, 1))
```

Asimismo, devuelven estos resultados:

* Primer ejemplo: la expresión es falsa, por lo que la función devuelve `true`.
* Segundo ejemplo: la expresión es verdadera, por lo que la función devuelve `false`.

<a name="or"></a>

### <a name="or"></a>or

Comprueba si al menos una expresión es true.
Devuelve true cuando al menos una expresión es verdadera, o bien devuelve false si las dos son falsas.

```
or(<expression1>, <expression2>)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*expresión1*>, <*expresión2*> | Sí | Boolean | Expresiones que se van a comprobar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| true o false | Boolean | Devuelve true cuando al menos una expresión es verdadera. Devuelve false si ambas expresiones son falsas. |
||||

*Ejemplo 1*

Estos ejemplos comprueban si al menos una expresión es verdadera:

```json
or(true, false)
or(false, false)
```

Asimismo, devuelven estos resultados:

* Primer ejemplo: al menos una expresión es verdadera, por lo que la función devuelve `true`.
* Segundo ejemplo: ambas expresiones son falsas, por lo que la función devuelve `false`.

*Ejemplo 2*

Estos ejemplos comprueban si al menos una expresión es verdadera:

```json
or(equals(1, 1), equals(1, 2))
or(equals(1, 2), equals(1, 3))
```

Asimismo, devuelven estos resultados:

* Primer ejemplo: al menos una expresión es verdadera, por lo que la función devuelve `true`.
* Segundo ejemplo: ambas expresiones son falsas, por lo que la función devuelve `false`.

<a name="rand"></a>

### <a name="rand"></a>rand

Devuelve un entero aleatorio de un intervalo especificado, que incluye solo el extremo inicial.

```
rand(<minValue>, <maxValue>)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*valor_mínimo*> | Sí | Entero | Entero más bajo del intervalo |
| <*valor_maximo*> | Sí | Entero | Entero que sigue al entero más alto del intervalo que puede devolver la función |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*random-result*> | Entero | Entero aleatorio devuelto del intervalo especificado |
||||

*Ejemplo*

Este ejemplo obtiene un entero aleatorio del intervalo especificado, sin incluir el valor máximo:

```
rand(1, 5)
```

Y devuelve uno de estos números como resultado: `1`, `2`, `3` o `4`

<a name="range"></a>

### <a name="range"></a>range

Devuelve una matriz de enteros que comienza en un entero especificado.

```
range(<startIndex>, <count>)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*startIndex*> | Sí | Entero | Valor entero que inicia la matriz como el primer elemento |
| <*recuento*> | Sí | Entero | Número de enteros de la matriz |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| [<*intervalo-resultado*>] | Array | Matriz de enteros comenzando a partir del índice especificado |
||||

*Ejemplo*

Este ejemplo crea una matriz de enteros que comienza en el índice especificado y tiene el número especificado de enteros:

```
range(1, 4)
```

Y devuelve este resultado: `[1, 2, 3, 4]`

<a name="replace"></a>

### <a name="replace"></a>replace

Reemplaza una subcadena por la cadena especificada y devuelve la cadena resultante. Esta función distingue mayúsculas de minúsculas.

```
replace('<text>', '<oldText>', '<newText>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena que contiene la subcadena que se va a reemplazar |
| <*oldText*> | Sí | String | Subcadena que se va a reemplazar. |
| <*newText*> | Sí | String | Cadena que se va a reemplazar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*updated-text*> | String | Cadena actualizada después de reemplazar la subcadena <p>Si no se encuentra la subcadena, devuelve la cadena original. |
||||

*Ejemplo*

Este ejemplo busca la subcadena "old" en "the old string" y reemplaza "old" por "new":

```
replace('the old string', 'old', 'new')
```

Y devuelve este resultado: `"the new string"`

<a name="skip"></a>

### <a name="skip"></a>skip

Quita elementos del principio de una colección y devuelve *todos los demás* elementos.

```
skip([<collection>], <count>)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*colección*> | Sí | Array | Colección cuyos elementos desea eliminar |
| <*recuento*> | Sí | Entero | Entero positivo para el número de elementos a eliminar al principio |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| [<*colección-actualizada*>] | Array | Colección actualizada después de eliminar los elementos especificados |
||||

*Ejemplo*

Este ejemplo elimina un elemento, el número 0, del principio de la matriz especificada:

```
skip(createArray(0, 1, 2, 3), 1)
```

Y devuelve esta matriz con los elementos restantes: `[1,2,3]`

<a name="split"></a>

### <a name="split"></a>split

Devuelve una matriz que contiene subcadenas, separadas por comas, en función de un carácter delimitador especificado en la cadena original.

```
split('<text>', '<delimiter>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | La cadena para separar en subcadenas según el delimitador especificado en la cadena original |
| <*delimitador*> | Sí | String | El carácter de la cadena original que se usará como delimitador |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| [<*subcadena1*>, <*subcadena2*>,...] | Array | Una matriz que contiene subcadenas de la cadena original, separadas por comas |
||||

*Ejemplo*

En este ejemplo se crea una matriz con las subcadenas de la cadena especificada según el carácter especificado como delimitador:

```
split('a_b_c', '_')
```

Y devuelve esta matriz como resultado: `["a","b","c"]`

<a name="startOfDay"></a>

### <a name="startofday"></a>startOfDay

Devuelve el inicio del día de una marca de tiempo.

```
startOfDay('<timestamp>', '<format>'?)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*formato*> | No | String | Puede ser un [especificador de formato sencillo](/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo especificada, pero a partir de la marca de hora cero del día |
||||

*Ejemplo*

Este ejemplo busca el inicio del día de esta marca de tiempo:

```
startOfDay('2018-03-15T13:30:30Z')
```

Y devuelve este resultado: `"2018-03-15T00:00:00.0000000Z"`

<a name="startOfHour"></a>

### <a name="startofhour"></a>startOfHour

Devuelve el inicio de la hora de una marca de tiempo.

```
startOfHour('<timestamp>', '<format>'?)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*formato*> | No | String | Puede ser un [especificador de formato sencillo](/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo especificada, pero a partir de la marca de minuto cero de la hora |
||||

*Ejemplo*

Este ejemplo busca el inicio de la hora de esta marca de tiempo:

```
startOfHour('2018-03-15T13:30:30Z')
```

Y devuelve este resultado: `"2018-03-15T13:00:00.0000000Z"`

<a name="startOfMonth"></a>

### <a name="startofmonth"></a>startOfMonth

Devuelve el inicio del mes de una marca de tiempo.

```
startOfMonth('<timestamp>', '<format>'?)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*formato*> | No | String | Puede ser un [especificador de formato sencillo](/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo especificada, pero a partir del primer día del mes en la marca de hora cero |
||||

*Ejemplo*

Este ejemplo devuelve el inicio del mes de esta marca de tiempo:

```
startOfMonth('2018-03-15T13:30:30Z')
```

Y devuelve este resultado: `"2018-03-01T00:00:00.0000000Z"`

<a name="startswith"></a>

### <a name="startswith"></a>startsWith

Comprueba si una cadena comienza con una subcadena especificada.
Devuelve true cuando se encuentra la subcadena o devuelve false cuando no se encuentra.
Esta función no distingue mayúsculas de minúsculas.

```
startsWith('<text>', '<searchText>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena que se va a comprobar |
| <*texto_a_buscar*> | Sí | String | Cadena inicial que se va a buscar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| true o false  | Boolean | Devuelve true cuando se encuentra la subcadena inicial. Devuelve false si no se encuentra. |
||||

*Ejemplo 1*

Este ejemplo comprueba si la cadena "hello world" comienza con la subcadena "hello":

```
startsWith('hello world', 'hello')
```

Y devuelve este resultado: `true`

*Ejemplo 2*

Este ejemplo comprueba si la cadena "hello world" comienza con la subcadena "greetings":

```
startsWith('hello world', 'greetings')
```

Y devuelve este resultado: `false`

<a name="string"></a>

### <a name="string"></a>string

Devuelve la versión de cadena de un valor.

```
string(<value>)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | Any | Valor que se va a convertir |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*valor-de-cadena*> | String | Versión de cadena del valor especificado |
||||

*Ejemplo 1*

Este ejemplo crea la versión de cadena de este número:

```
string(10)
```

Y devuelve este resultado: `"10"`

*Ejemplo 2*

Este ejemplo crea una cadena a partir del objeto JSON especificado y usa el carácter de barra diagonal inversa (\\) como carácter de escape para la marca de comillas dobles (").

```
string( { "name": "Sophie Owen" } )
```

Y devuelve este resultado: `"{ \\"name\\": \\"Sophie Owen\\" }"`

<a name="sub"></a>

### <a name="sub"></a>sub

Devuelve el resultado de restar el segundo número del primero.

```
sub(<minuend>, <subtrahend>)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*minuendo*> | Sí | Integer o Float | Número del que se resta el *sustraendo* |
| <*sustraendo*> | Sí | Integer o Float | Número que se resta del *minuendo* |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*resultado*> | Integer o Float | Resultado de restar el segundo número del primero |
||||

*Ejemplo*

Este ejemplo resta el segundo número del primer número:

```
sub(10.3, .3)
```

Y devuelve este resultado: `10`

<a name="substring"></a>

### <a name="substring"></a>substring

Devuelve caracteres de una cadena a partir de la posición especificada o índice.
Los valores de índice comienzan desde el número 0.

```
substring('<text>', <startIndex>, <length>)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena cuyos caracteres se toman |
| <*startIndex*> | Sí | Entero | Un número positivo igual o mayor que 0 que desea utilizar como el valor de índice o la posición inicial |
| <*longitud*> | Sí | Entero | Número positivo de caracteres que desea incluir en la subcadena |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*subcadena-resultante*> | String | Subcadena con el número especificado de caracteres, empezando en la posición de índice especificada de la cadena de origen |
||||

*Ejemplo*

Este ejemplo crea una subcadena de cinco caracteres a partir de la cadena especificada, empezando en el valor de índice 6:

```
substring('hello world', 6, 5)
```

Y devuelve este resultado: `"world"`

<a name="subtractFromTime"></a>

### <a name="subtractfromtime"></a>subtractFromTime

Resta un número de unidades de tiempo de una marca de tiempo.
Consulte también [getPastTime](#getPastTime).

```
subtractFromTime('<timestamp>', <interval>, '<timeUnit>', '<format>'?)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*intervalo*> | Sí | Entero | Número de unidades de tiempo especificadas que se va a sustraer |
| <*unidad_de_tiempo*> | Sí | String | La unidad de tiempo que se usará con *intervalo*: "Segundo", "Minuto", "Hora", "Día", "Semana", "Mes", "Año" |
| <*formato*> | No | String | Puede ser un [especificador de formato sencillo](/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo menos el número de unidades de tiempo especificado |
||||

*Ejemplo 1*

Este ejemplo resta un día de esta marca de tiempo:

```
subtractFromTime('2018-01-02T00:00:00Z', 1, 'Day')
```

Y devuelve este resultado: `"2018-01-01T00:00:00:0000000Z"`

*Ejemplo 2*

Este ejemplo resta un día de esta marca de tiempo:

```
subtractFromTime('2018-01-02T00:00:00Z', 1, 'Day', 'D')
```

Y devuelve el resultado con el formato "D" opcional: `"Monday, January, 1, 2018"`

<a name="take"></a>

### <a name="take"></a>take

Devuelve elementos del principio de una colección.

```
take('<collection>', <count>)
take([<collection>], <count>)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*colección*> | Sí | Cadena o matriz | Colección cuyos elementos desea tomar |
| <*recuento*> | Sí | Entero | Entero positivo para el número de elementos a tomar desde el principio |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*subconjunto*> o [<*subconjunto*>] | Cadena o matriz, respectivamente | Cadena o matriz que tiene el número especificado de elementos tomados desde el principio de la colección original |
||||

*Ejemplo*

Estos ejemplos obtienen el número especificado de elementos desde el principio de estas colecciones:

```
take('abcde', 3)
take(createArray(0, 1, 2, 3, 4), 3)
```

Asimismo, devuelven estos resultados:

* Primer ejemplo: `"abc"`
* Segundo ejemplo: `[0, 1, 2]`

<a name="ticks"></a>

### <a name="ticks"></a>ticks

Devuelve el valor de la propiedad `ticks` de una marca de tiempo especificada.
Un *tick* es un intervalo de 100 nanosegundos.

```
ticks('<timestamp>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena de una marca de tiempo |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*número-de-ticks*> | Entero | Número de ticks desde la marca de tiempo especificada |
||||

<a name="toLower"></a>

### <a name="tolower"></a>toLower

Devuelve una cadena en formato de minúsculas. Si un carácter de la cadena no tiene un equivalente en minúsculas, se incluye sin cambios en la cadena devuelta.

```
toLower('<text>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena que se devuelve en formato de minúsculas |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*texto-en-minúsculas*> | String | Cadena original en minúsculas |
||||

*Ejemplo*

Este ejemplo convierte esta cadena a minúsculas:

```
toLower('Hello World')
```

Y devuelve este resultado: `"hello world"`

<a name="toUpper"></a>

### <a name="toupper"></a>toUpper

Devuelve una cadena en formato de mayúsculas. Si un carácter de la cadena no tiene un equivalente en mayúsculas, se incluye sin cambios en la cadena devuelta.

```
toUpper('<text>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena que se devuelve en formato de mayúsculas |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*texto-en-mayúsculas*> | String | Cadena original en mayúsculas |
||||

*Ejemplo*

Este ejemplo convierte esta cadena a mayúsculas:

```
toUpper('Hello World')
```

Y devuelve este resultado: `"HELLO WORLD"`

<a name="trim"></a>

### <a name="trim"></a>trim

Quita el espacio en blanco inicial y final de una cadena y devuelve la cadena actualizada.

```
trim('<text>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena que contiene los espacios en blanco iniciales y finales que se van a eliminar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*texto_actualizado*> | String | Versión actualizada de la cadena original sin espacios en blanco iniciales o finales |
||||

*Ejemplo*

Este ejemplo quita el espacio en blanco inicial y final de la cadena " Hello World ":

```
trim(' Hello World  ')
```

Y devuelve este resultado: `"Hello World"`

<a name="union"></a>

### <a name="union"></a>union

Devuelve una colección que tiene *todos* los elementos de las colecciones especificadas.
Para que aparezca en el resultado, un elemento debe aparecer en alguna de las colecciones que se pasan a esta función. Si uno o más elementos tienen el mismo nombre, el último elemento con ese nombre aparece en el resultado.

```
union('<collection1>', '<collection2>', ...)
union([<collection1>], [<collection2>], ...)
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*colección1*>, <*colección2*>, ...  | Sí | Matriz u objeto, pero no ambos | Colecciones de las que desean *todos* los elementos |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*colección_actualizada*> | Matriz u objeto, respectivamente | Colección con todos los elementos de las colecciones especificadas, sin duplicados. |
||||

*Ejemplo*

Este ejemplo obtiene *todos* los elementos de estas colecciones:

```
union(createArray(1, 2, 3), createArray(1, 2, 10, 101))
```

Y devuelve este resultado: `[1, 2, 3, 10, 101]`

<a name="uriComponent"></a>

### <a name="uricomponent"></a>uriComponent

Devuelve una versión codificada de un identificador uniforme de recursos (URI) de una cadena reemplazando los caracteres no seguros de la dirección URL por caracteres de escape.
Use esta función en lugar de [encodeUriComponent()](#encodeUriComponent).
Aunque ambas funciones funcionan del mismo modo, `uriComponent()` es preferible.

```
uriComponent('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Cadena que se va a convertir en formato codificado de URI |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*uri-codificado*> | String | Cadena codificada en formato URI con caracteres de escape |
||||

*Ejemplo*

Este ejemplo crea una versión codificada en formato URI de esta cadena:

```
uriComponent('https://contoso.com')
```

Y devuelve este resultado: `"http%3A%2F%2Fcontoso.com"`

<a name="uriComponentToBinary"></a>

### <a name="uricomponenttobinary"></a>uriComponentToBinary

Devuelve la versión binaria de un componente de un identificador uniforme de recursos (URI).

```
uriComponentToBinary('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Cadena con codificación URI que se va a convertir |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*binario-de-codificado-uri*> | String | Versión binaria de la cadena con codificación URI. El contenido binario está codificado en base64 y representado por `$content`. |
||||

*Ejemplo*

Este ejemplo crea la versión binaria de esta cadena con codificación URI:

```
uriComponentToBinary('http%3A%2F%2Fcontoso.com')
```

Y devuelve este resultado:

`"001000100110100001110100011101000111000000100101001100
11010000010010010100110010010001100010010100110010010001
10011000110110111101101110011101000110111101110011011011
110010111001100011011011110110110100100010"`

<a name="uriComponentToString"></a>

### <a name="uricomponenttostring"></a>uriComponentToString

Devuelve la versión de cadena de una cadena codificada en formato de identificador uniforme de recursos (URI), decodificando dicha cadena.

```
uriComponentToString('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Cadena con codificación URI que se va a decodificar |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*uri-decodificado*> | String | Versión decodificada de la cadena con codificación URI |
||||

*Ejemplo*

Este ejemplo crea la versión de cadena decodificada de esta cadena con codificación URI:

```
uriComponentToString('http%3A%2F%2Fcontoso.com')
```

Y devuelve este resultado: `"https://contoso.com"`

<a name="utcNow"></a>

### <a name="utcnow"></a>utcNow

Devuelve la marca de tiempo actual.

```
utcNow('<format>')
```

Si lo desea, puede especificar un formato diferente con el parámetro <*format*>.

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*formato*> | No | String | Puede ser un [especificador de formato sencillo](/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actual*> | String | Fecha y hora actuales |
||||

*Ejemplo 1*

Imagine que hoy es 15 de abril de 2018 a las 1:00:00 P.M.
Este ejemplo obtiene la marca de tiempo actual:

```
utcNow()
```

Y devuelve este resultado: `"2018-04-15T13:00:00.0000000Z"`

*Ejemplo 2*

Imagine que hoy es 15 de abril de 2018 a las 1:00:00 P.M.
Este ejemplo obtiene la marca de tiempo actual con el formato "D" opcional:

```
utcNow('D')
```

Y devuelve este resultado: `"Sunday, April 15, 2018"`

<a name="xml"></a>

### <a name="xml"></a>Xml

Devuelve la versión XML de una cadena que contiene un objeto JSON.

```
xml('<value>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*value*> | Sí | String | Cadena con el objeto JSON que se va a convertir <p>El objeto JSON debe tener solo una propiedad raíz, que no puede ser una matriz. <br>Use el carácter de barra diagonal inversa (\\) como carácter de escape para la marca de comillas dobles ("). |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*versión-xml*> | Object | XML codificado de la cadena u objeto JSON especificados |
||||

*Ejemplo 1*

Este ejemplo crea la versión XML de esta cadena, que contiene un objeto JSON:

`xml(json('{ \"name\": \"Sophia Owen\" }'))`

Y devuelve este código XML resultante:

```xml
<name>Sophia Owen</name>
```

*Ejemplo 2*

Suponga que tiene este objeto JSON:

```json
{
  "person": {
    "name": "Sophia Owen",
    "city": "Seattle"
  }
}
```

Este ejemplo crea código XML de una cadena que contiene este objeto JSON:

`xml(json('{\"person\": {\"name\": \"Sophia Owen\", \"city\": \"Seattle\"}}'))`

Y devuelve este código XML resultante:

```xml
<person>
  <name>Sophia Owen</name>
  <city>Seattle</city>
<person>
```

<a name="xpath"></a>

### <a name="xpath"></a>xpath

Comprueba el código XML de los nodos o valores que coinciden con una expresión XPath (XML Path Language) y devuelve los nodos o valores coincidentes. Una expresión XPath o simplemente "XPath", le ayuda a recorrer una estructura de documento XML para que pueda seleccionar nodos o valores de proceso del contenido XML.

```
xpath('<xml>', '<xpath>')
```

| Parámetro | Obligatorio | Tipo | Descripción |
| --------- | -------- | ---- | ----------- |
| <*xml*> | Sí | Any | Cadena XML en la que buscar nodos o valores que coincidan con un valor de la expresión XPath |
| <*xpath*> | Sí | Any | Expresión XPath utilizada para buscar nodos XML o valores coincidentes |
|||||

| Valor devuelto | Tipo | Descripción |
| ------------ | ---- | ----------- |
| <*nodo-xml*> | XML | Nodo XML si solo un nodo coincide con la expresión XPath especificada |
| <*value*> | Any | Valor de un nodo XML si solo un valor coincide con la expresión XPath especificada |
| [<*xml-nodo1*>, <*xml-nodo2*>, ...] </br>O bien </br>[<*valor1*>, <*valor2*>, ...] | Array | Matriz con los nodos XML o valores que coinciden con la expresión XPath especificada |
||||

*Ejemplo 1*

Siguiendo el ejemplo 1, este ejemplo busca nodos que coincidan con el nodo `<count></count>` y agrega los valores de nodo con la función `sum()`:

`xpath(xml(parameters('items')), 'sum(/produce/item/count)')`

Y devuelve este resultado: `30`

*Ejemplo 2*

En este ejemplo, ambas expresiones buscan nodos que coincidan con el nodo `<location></location>` de los argumentos especificados, que incluye código XML con un espacio de nombres. Las expresiones utilizan el carácter de barra diagonal inversa (\\) como carácter de escape para la marca de comillas dobles (").

* *Expresión 1*

  `xpath(xml(body('Http')), '/*[name()=\"file\"]/*[name()=\"location\"]')`

* *Expresión 2*

  `xpath(xml(body('Http')), '/*[local-name()=\"file\" and namespace-uri()=\"http://contoso.com\"]/*[local-name()=\"location\"]')`

Estos son los argumentos:

* Este código XML, que incluye el espacio de nombres del documento XML, `xmlns="http://contoso.com"`:

  ```xml
  <?xml version="1.0"?> <file xmlns="http://contoso.com"> <location>Paris</location> </file>
  ```

* Cualquier expresión XPath:

  * `/*[name()=\"file\"]/*[name()=\"location\"]`

  * `/*[local-name()=\"file\" and namespace-uri()=\"http://contoso.com\"]/*[local-name()=\"location\"]`

Este es el nodo de resultados que coincide con el nodo `<location></location>`:

```xml
<location xmlns="https://contoso.com">Paris</location>
```

*Ejemplo 3*

Siguiendo el ejemplo 3, este ejemplo busca el valor en el nodo `<location></location>`:

`xpath(xml(body('Http')), 'string(/*[name()=\"file\"]/*[name()=\"location\"])')`

Y devuelve este resultado: `"Paris"`

## <a name="next-steps"></a>Pasos siguientes
Para obtener una lista de las variables del sistema que se pueden usar en las expresiones, vea [Variables del sistema](control-flow-system-variables.md).
