---
title: Referencia del lenguaje SQL de aceleración de consultas
titleSuffix: Azure Storage
description: Aprenda a usar la sintaxis SQL de aceleración de consultas.
services: storage
author: normesta
ms.service: storage
ms.topic: conceptual
ms.date: 09/09/2020
ms.author: normesta
ms.subservice: data-lake-storage-gen2
ms.reviewer: ereilebr
ms.openlocfilehash: f696a6b071d353c98e87387d5640e35ff579460e
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110477816"
---
# <a name="query-acceleration-sql-language-reference"></a>Referencia del lenguaje SQL de aceleración de consultas

La aceleración de consultas es compatible con un lenguaje ANSI de tipo SQL para expresar consultas sobre el contenido de blobs.  El dialecto SQL de aceleración de consultas es un subconjunto de SQL ANSI, con un conjunto limitado de tipos de datos admitidos, operadores, etc., pero también se expande en SQL ANSI para admitir consultas en formatos de datos semiestructurados jerárquicos, como JSON. 

## <a name="select-syntax"></a>Sintaxis de SELECT

La única instrucción SQL que admite la aceleración de consultas es la instrucción SELECT. Este ejemplo devuelve todas las filas para las que la expresión devuelve true.

```sql
SELECT * FROM table [WHERE expression] [LIMIT limit]
```

En el caso de los datos con formato CSV, *table* debe ser `BlobStorage`.  Esto significa que la consulta se ejecutará en cualquier blob que se haya especificado en la llamada REST.
En el caso de los datos con formato JSON, *table* es un "descriptor de tabla".   Consulte la sección [Descriptores de tabla](#table-descriptors) de este artículo.

En el ejemplo siguiente, para cada fila para la que la *expresión* WHERE devuelva true, esta instrucción devolverá una nueva fila que se realiza a partir de la evaluación de cada una de las expresiones de proyección.


```sql
SELECT expression [, expression …] FROM table [WHERE expression] [LIMIT limit]
```

Puede especificar una o varias columnas específicas como parte de la expresión SELECT (por ejemplo: `SELECT Title, Author, ISBN`). 

> [!NOTE]
> El número máximo de columnas específicas que puede usar en la expresión SELECT es 49. Si necesita que la instrucción SELECT devuelva más de 49 columnas, use un carácter comodín (`*`) para la expresión SELECT (por ejemplo: `SELECT *`). 

En el ejemplo siguiente se devuelve un cálculo de agregado (por ejemplo, el valor medio de una columna determinada) sobre cada una de las filas para las que la *expresión* devuelve true. 

```sql
SELECT aggregate_expression FROM table [WHERE expression] [LIMIT limit]
```

En el ejemplo siguiente se devuelven los desplazamientos adecuados para dividir un blob con formato CSV.  Consulte la sección [Sys.Split](#sys-split) de este artículo.

```sql
SELECT sys.split(split_size)FROM BlobStorage
```

<a id="data-types"></a>

## <a name="data-types"></a>Tipo de datos

|Tipo de datos|Descripción|
|---------|-------------------------------------------|
|INT      |Entero con signo de 64 bits.                     |
|FLOAT    |Punto flotante de 64 bits ("precisión doble").|
|STRING   |Cadena Unicode de longitud variable.            |
|timestamp|A un momento dado.                           |
|BOOLEAN  |True o false.                             |

Al leer valores de datos con formato CSV, todos los valores se leen como cadenas.  Los valores de cadena se pueden convertir a otros tipos mediante expresiones CAST.  Los valores se pueden convertir implícitamente a otros tipos en función del contexto. Para obtener más información, consulte [Prioridad de tipo de datos (Transact-SQL)](/sql/t-sql/data-types/data-type-precedence-transact-sql).

## <a name="expressions"></a>Expresiones

### <a name="referencing-fields"></a>Campos de referencia

En el caso de los datos con formato JSON o los datos con formato CSV con una fila de encabezado, es posible hacer referencia a los campos por su nombre.  Los nombres de campos pueden ir entre comillas o sin comillas. Los nombres de campo entre comillas se incluyen entre caracteres de comillas dobles ("), pueden contener espacios y distinguen mayúsculas de minúsculas.  Los nombres de campo sin comillas no distinguen mayúsculas de minúsculas y no pueden contener caracteres especiales.

En los datos con formato CSV, también se puede hacer referencia a los campos por ordinal, precedidos por un carácter de subrayado (_).  Por ejemplo, se puede hacer referencia al primer campo como _1 o al undécimo campo como _11.  La referencia a campos por ordinal es útil para los datos con formato CSV que no contienen una fila de encabezado, en cuyo caso la única manera de hacer referencia a un campo determinado es por ordinal.

### <a name="operators"></a>Operadores

Se admiten los siguientes operadores de SQL estándar:

|Operator|Descripción|
|--|--|
|[=](/sql/t-sql/language-elements/equals-transact-sql)    |Compara la equivalencia de dos expresiones (es un operador de comparación).|
|[!=](/sql/t-sql/language-elements/not-equal-to-transact-sql-exclamation)    |Prueba si una expresión es distinta de otra expresión (es un operador de comparación).|
|[<>](/sql/t-sql/language-elements/not-equal-to-transact-sql-traditional)    |Compara dos expresiones para "no igual que" (un operador de comparación).|
|[<](/sql/t-sql/language-elements/less-than-transact-sql)    |Compara dos expresiones para "menor que" (un operador de comparación).|
|[<=](/sql/t-sql/language-elements/less-than-or-equal-to-transact-sql)    |Compara dos expresiones para "menor o igual que" (un operador de comparación).|
|[>](/sql/t-sql/language-elements/greater-than-transact-sql)    |Compara dos expresiones para "mayor que" (un operador de comparación). |
|[>=](/sql/t-sql/language-elements/greater-than-or-equal-to-transact-sql)    |Compara dos expresiones para "mayor o igual que" (un operador de comparación).|
|[+](/sql/t-sql/language-elements/add-transact-sql)    |Suma dos números. Este operador aritmético de suma también puede sumar un número, en días, a una fecha.|
|[-](/sql/t-sql/language-elements/subtract-transact-sql)    |Resta dos números (un operador aritmético de sustracción). |
|[/](/sql/t-sql/language-elements/divide-transact-sql)    |Divide un número entre otro (es un operador aritmético de división).|
|[*](/sql/t-sql/language-elements/multiply-transact-sql)    |Multiplica dos expresiones (es un operador aritmético de multiplicación).|
|[%](/sql/t-sql/language-elements/modulo-transact-sql)    |Devuelve el resto de un número dividido entre otro.|
|[AND](/sql/t-sql/language-elements/bitwise-and-transact-sql)    |Realiza una operación lógica AND bit a bit entre dos valores enteros.|
|[OR](/sql/t-sql/language-elements/bitwise-or-transact-sql)    |Realiza una operación OR bit a bit lógica entre dos valores de tipo entero especificados tal y como aparecen traducidos en expresiones binarias en instrucciones Transact-SQL.|
|[NOT](/sql/t-sql/language-elements/not-transact-sql)    |Niega la entrada de un valor booleano.|
|[CAST](/sql/t-sql/functions/cast-and-convert-transact-sql)    |Convierte una expresión de un tipo de datos a otro.|
|[BETWEEN](/sql/t-sql/language-elements/between-transact-sql)    |Especifica un intervalo que se va a probar.|
|[IN](/sql/t-sql/language-elements/in-transact-sql)    |Determina si un valor especificado coincide con algún valor de una subconsulta o una lista.|
|[NULLIF](/sql/t-sql/language-elements/nullif-transact-sql)    |Devuelve un valor NULL si las dos expresiones especificadas son iguales.|
|[COALESCE](/sql/t-sql/language-elements/coalesce-transact-sql)    |Evalúa los argumentos en orden y devuelve el valor actual de la primera expresión que inicialmente no se evalúa como NULL.|

Si los tipos de datos a la izquierda y a la derecha de un operador son diferentes, la conversión automática se realizará según las reglas especificadas aquí: [Prioridad de tipo de datos (Transact-SQL)](/sql/t-sql/data-types/data-type-precedence-transact-sql).

El lenguaje SQL de aceleración de consultas solo admite un subconjunto muy pequeño de los tipos de datos descritos en ese artículo.  Consulte la sección [Tipos de datos](#data-types) de este artículo.

### <a name="casts"></a>Conversiones

El lenguaje SQL de aceleración de consultas es compatible con el operador CAST según las reglas siguientes: [Conversión de tipos de datos (motor de base de datos)](/sql/t-sql/data-types/data-type-conversion-database-engine).  

El lenguaje SQL de aceleración de consultas solo admite un subconjunto minúsculo de los tipos de datos descritos en ese artículo.  Consulte la sección [Tipos de datos](#data-types) de este artículo.

### <a name="string-functions"></a>Funciones de cadena

El lenguaje SQL de aceleración de consultas admite las siguientes funciones de cadena de SQL estándar:

|Función|Descripción|
|--|--|
|CHAR_LENGTH    | Devuelve la longitud en caracteres de la expresión de cadena, si la expresión de cadena es de un tipo de datos de caracteres; de lo contrario, devuelve la longitud en bytes de la expresión de cadena (el entero más pequeño no es menor que el número de bits dividido entre 8). (Esta función es igual que la función CHARACTER_LENGTH).|
|CHARACTER_LENGTH    |Devuelve la longitud en caracteres de la expresión de cadena, si la expresión de cadena es de un tipo de datos de caracteres; de lo contrario, devuelve la longitud en bytes de la expresión de cadena (el entero más pequeño no es menor que el número de bits dividido entre 8). (Esta función es igual que la función CHAR_LENGTH).|
|[LOWER](/sql/t-sql/functions/lower-transact-sql)    |Devuelve una expresión de caracteres después de convertir en minúsculas los datos de caracteres en mayúsculas.|
|[UPPER](/sql/t-sql/functions/upper-transact-sql)    |Devuelve una expresión de caracteres con datos de caracteres en minúsculas convertidos a mayúsculas.|
|[SUBSTRING](/sql/t-sql/functions/substring-transact-sql)    |Devuelve parte de una expresión de caracteres, binaria, de texto o de imagen en SQL Server.|
|[TRIM](/sql/t-sql/functions/trim-transact-sql)    |Esto permite quitar el carácter de espacio char(32) u otros caracteres especificados del principio y del final de una cadena.|
|LEADING    |Quita el carácter de espacio char(32) u otros caracteres especificados del principio de una cadena.|
|TRAILING    |Quita el carácter de espacio char(32) u otros caracteres especificados del final de una cadena.|

Estos son algunos ejemplos:

|Función|Ejemplo|Resultado|
|---|---|---|
|CHARACTER_LENGTH|``SELECT CHARACTER_LENGTH('abcdefg') from BlobStorage`` |``7``|
|CHAR_LENGTH|``SELECT CHAR_LENGTH(_1) from BlobStorage``|``1``|
|LOWER|``SELECT LOWER('AbCdEfG') from BlobStorage``|``abcdefg``|
|UPPER|``SELECT UPPER('AbCdEfG') from BlobStorage``|``ABCDEFG``|
|SUBSTRING|``SUBSTRING('123456789', 1, 5)``|``23456``|
|TRIM|``TRIM(BOTH '123' FROM '1112211Microsoft22211122')``|``Microsoft``|

### <a name="date-functions"></a>Funciones de fecha

Se admiten las funciones de datos de SQL estándar siguientes:

``DATE_ADD``, ``DATE_DIFF``, ``EXTRACT``, ``TO_STRING``, ``TO_TIMESTAMP``.

Actualmente, se convierten todos los [formatos de fecha del estándar IS08601](https://www.w3.org/TR/NOTE-datetime). 

#### <a name="date_add-function"></a>Función DATE_ADD

El lenguaje SQL de aceleración de consultas admite año, mes, día, hora, minuto y segundo para la función ``DATE_ADD``.

Ejemplos:

``sql DATE_ADD(datepart, quantity, timestamp) DATE_ADD('minute', 1, CAST('2017-01-02T03:04:05.006Z' AS TIMESTAMP)
```

#### DATE_DIFF function

The query acceleration SQL language supports year, month, day, hour, minute, second for the ``DATE_DIFF`` function.

```sql
DATE_DIFF(datepart, timestamp, timestamp)
DATE_DIFF('hour','2018-11-09T00:00+05:30','2018-11-09T01:00:23-08:00') 
```

#### <a name="extract-function"></a>Función EXTRACT

En el caso de EXTRACT, además de la parte de la fecha compatible con la función ``DATE_ADD``, el lenguaje SQL de aceleración de consultas admite timezone_hour y timezone_minute como parte de la fecha.

Ejemplos:

```sql
EXTRACT(datepart FROM timestampstring)
EXTRACT(YEAR FROM '2010-01-01T')
```

#### <a name="to_string-function"></a>Función TO_STRING

Ejemplos:

```sql
TO_STRING(TimeStamp , format)
TO_STRING(CAST('1969-07-20T20:18Z' AS TIMESTAMP),  'MMMM d, y')
```

En esta tabla se describen las cadenas que se pueden usar para especificar el formato de salida de la función ``TO_STRING``.

|Cadena de formato    |Output                               |
|-----------------|-------------------------------------|
|yy               |Año con formato de dos dígitos (1999 como "99")|
|y                |Año con formato de cuatro dígitos               |
|aaaa             |Año con formato de cuatro dígitos               |
|M                |Mes del año (1)                    |
|MM               |Mes completado con cero (01)               |
|MMM              |Mes del año abreviado (ENE)            |
|MMMM             |Mes completo: mayo                      |
|d                |Día del mes (1-31)                  |
|dd               |Día del mes de completado con cero (01-31)     |
|a                |a. m. o p. m.                             |
|h                |Hora del día (1-12)                   |
|hh               |Horas del día completadas con cero (01-12)     |
|H                |Hora del día (0-23)                   |
|HH               |Hora del día completada con cero (00-23)      |
|m                |Minuto de la hora (0-59)                |
|MM               |Minuto completado con cero (00-59)           |
|s                |Segundo de minutos (0-59)             |
|ss               |Segundos completados con cero (00-59)          |
|S                |Fracción de segundos (0,1-0,9)        |
|SS               |Fracción de segundos (0,01-0,99)      |
|SSS              |Fracción de segundos (0,001-0,999)    |
|X                |Desplazamiento en horas                      |
|XX o XXXX       |Desplazamiento en horas y minutos (+0430)  |
|XXX o XXXXX     |Desplazamiento en horas y minutos (-07:00) |
|x                |Desplazamiento en horas (7)                  |
|xx o xxxx       |Desplazamiento en hora y minuto (+0530)    |
|Xxx o xxxxx     |Desplazamiento en hora y minuto (+05:30)   |

#### <a name="to_timestamp-function"></a>Función TO_TIMESTAMP

Solo se admiten los formatos IS08601.

Ejemplos:

```sql
TO_TIMESTAMP(string)
TO_TIMESTAMP('2007T')
```

> [!NOTE]
> También puede usar la función ``UTCNOW`` para obtener la hora del sistema.


## <a name="aggregate-expressions"></a>Expresiones de agregado

Una instrucción SELECT puede contener una o varias expresiones de proyección o una sola expresión de agregado.  Se admiten las siguientes expresiones de agregado:

|Expression|Descripción|
|--|--|
|[COUNT(\*)](/sql/t-sql/functions/count-transact-sql)    |Devuelve el número de registros que coinciden con la expresión de predicado.|
|[COUNT(expression)](/sql/t-sql/functions/count-transact-sql)    |Devuelve el número de registros cuya expresión no es NULL.|
|[AVERAGE(expression)](/sql/t-sql/functions/avg-transact-sql)    |Devuelve el promedio de los valores no NULL de la expresión.|
|[MIN(expression)](/sql/t-sql/functions/min-transact-sql)    |Devuelve el valor mínimo no NULL de la expresión.|
|[MAX(expression](/sql/t-sql/functions/max-transact-sql)    |Devuelve el valor máximo no NULL de la expresión.|
|[SUM(expression)](/sql/t-sql/functions/sum-transact-sql)    |Devuelve la suma de todos los valores no NULL de la expresión.|

### <a name="missing"></a>MISSING

El operador ``IS MISSING`` es el único no estándar que admite el lenguaje SQL de aceleración de consultas.  En el caso de los datos JSON, si falta un campo de un registro de entrada determinado, el campo de expresión ``IS MISSING`` se evaluará como el valor booleano true.

<a id="table-descriptors"></a>

## <a name="table-descriptors"></a>Descriptores de tabla

En el caso de los datos CSV, el nombre de la tabla siempre es `BlobStorage`.  Por ejemplo:

```sql
SELECT * FROM BlobStorage
```

En el caso de los datos JSON, hay disponibles opciones adicionales:

```sql
SELECT * FROM BlobStorage[*].path
```

Esta opción permite realizar consultas en subconjuntos de los datos JSON.

En el caso de las consultas JSON, puede mencionar la ruta de acceso en parte de la cláusula FROM. Estas rutas de acceso le ayudarán a analizar el subconjunto de datos JSON. Estas rutas de acceso pueden hacer referencia a valores de objeto y matriz JSON.

Facilitemos un ejemplo para comprender esto con más detalle.

Estos son los datos de ejemplo:

```json
{
  "id": 1,
  "name": "mouse",
  "price": 12.5,
  "tags": [
    "wireless",
    "accessory"
  ],
  "dimensions": {
    "length": 3,
    "width": 2,
    "height": 2
  },
  "weight": 0.2,
  "warehouses": [
    {
      "latitude": 41.8,
      "longitude": -87.6
    }
  ]
}
```

Puede que le interese solo el objeto JSON `warehouses` de los datos anteriores. El objeto `warehouses` es un tipo de matriz JSON, por lo que puede mencionarlo en la cláusula FROM. La consulta de ejemplo puede tener un aspecto similar al siguiente.

```sql
SELECT latitude FROM BlobStorage[*].warehouses[*]
```

La consulta obtiene todos los campos, pero selecciona solo la latitud.

Si quiere tener acceso solo al valor del objeto JSON `dimensions`, puede usar la referencia a ese objeto en la consulta. Por ejemplo:

```sql
SELECT length FROM BlobStorage[*].dimensions
```

Esto también limita el acceso a los miembros del objeto `dimensions`. Si quiere tener acceso a otros miembros de campos JSON y valores internos de objetos JSON, puede usar una consulta como la que se muestra en el ejemplo siguiente:

```sql
SELECT weight,warehouses[0].longitude,id,tags[1] FROM BlobStorage[*]
```

> [!NOTE]
> BlobStorage y BlobStorage[\*] hacen referencia a todo el objeto. Sin embargo, si tiene una ruta de acceso en la cláusula FROM, deberá usar BlobStorage[\*].path

<a id="sys-split"></a>

## <a name="syssplit"></a>Sys.Split

Se trata de una forma especial de la instrucción SELECT, que solo está disponible para los datos con formato CSV.

```sql
SELECT sys.split(split_size)FROM BlobStorage
```

Utilice esta instrucción en los casos en los que quiere descargar y después procesar registros de datos CSV en lotes. De este modo, puede procesar registros en paralelo en lugar de tener que descargar todos los registros al mismo tiempo. Esta instrucción no devuelve registros del archivo CSV. En su lugar, devuelve una colección de tamaños de lote. Después, puede usar cada tamaño de lote para recuperar un lote de registros de datos. 

Use el parámetro *split_size* para especificar el número de bytes que quiere que contenga cada lote. Por ejemplo, si quiere procesar solo 10 MB de datos a la vez, la instrucción tiene el siguiente aspecto: `SELECT sys.split(10485760)FROM BlobStorage`, porque 10 MB es igual a 10 485 760 bytes. Cada lote contendrá tantos registros como quepan en esos 10 MB. 

En la mayoría de los casos, el tamaño de cada lote será ligeramente mayor que el número especificado. Esto se debe a que un lote no puede contener un registro parcial. Si el último registro de un lote comienza antes del final del umbral, el lote será mayor para que pueda contener el registro completo. El tamaño del último lote probablemente será menor que el tamaño que especifique.

>[!NOTE]
> El valor de split_size debe ser de al menos 10 MB (10485760).

## <a name="see-also"></a>Consulte también

- [Aceleración de consultas de Azure Data Lake Storage](data-lake-storage-query-acceleration.md)
- [Filtro de datos mediante la aceleración de consultas de Azure Data Lake Storage](data-lake-storage-query-acceleration-how-to.md)