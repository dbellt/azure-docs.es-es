---
title: Funciones de limpieza y transformación de datos en Azure Data Factory
description: Introducción a las funciones disponibles de limpieza y transformación de datos en Azure Data Factory
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.date: 04/16/2021
ms.openlocfilehash: f7a4041d87e00fa01ae5ae4dff0cade3b9755d31
ms.sourcegitcommit: 950e98d5b3e9984b884673e59e0d2c9aaeabb5bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2021
ms.locfileid: "107600947"
---
# <a name="transformation-functions-in-power-query-for-data-wrangling"></a>Funciones de transformación en Power Query para la limpieza y transformación de datos

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

La limpieza y transformación de datos en Azure Data Factory permite la ágil preparación, y limpieza y transformación de datos sin código a escala de nube mediante la traducción de scripts ```M``` de Power Query al script de Data Flow. ADF se integra en [Power Query online](/powerquery-m/power-query-m-reference) y pone las funciones ```M``` de Power Query a disposición para la limpieza y transformación de datos a través de la ejecución de Spark con la infraestructura de Spark de flujo de datos. 

> [!NOTE]
> Power Query en ADF está actualmente disponible en versión preliminar pública

Actualmente no se admiten todas las funciones de Power Query M para la limpieza y transformación de datos, a pesar de estar disponibles durante la creación. Al compilar las recopilaciones, aparecerá el siguiente mensaje de error si no se admite una función:

`UserQuery : Expression.Error: The transformation logic is not supported as it requires dynamic access to rows of data, which cannot be scaled out.`

A continuación se muestra una lista de funciones admitidas de Power Query M.

## <a name="column-management"></a>Administración de columnas

* Selección: [Table.SelectColumns](/powerquery-m/table-selectcolumns)
* Eliminación: [Table.RemoveColumns](/powerquery-m/table-removecolumns)
* Cambio de nombre: [Table.RenameColumns](/powerquery-m/table-renamecolumns), [Table.PrefixColumns](/powerquery-m/table-prefixcolumns), [Table.TransformColumnNames](/powerquery-m/table-transformcolumnnames)
* Reordenación: [Table.ReorderColumns](/powerquery-m/table-reordercolumns)

## <a name="row-filtering"></a>Filtrado de filas

Use la función de M [Table.SelectRows](/powerquery-m/table-selectrows) para filtrar según las condiciones siguientes:

* Igualdad y desigualdad
* Comparaciones numéricas, de texto y de fechas (pero no de fecha y hora)
* Información numérica, como [Number.IsEven](/powerquery-m/number-iseven)/[Odd](/powerquery-m/number-iseven)
* Contención de texto mediante [Text.Contains](/powerquery-m/text-contains), [Text.StartsWith](/powerquery-m/text-startswith) o [Text.EndsWith](/powerquery-m/text-endswith)
* Intervalos de fechas (incluidas todas las [funciones de fecha](/powerquery-m/date-functions) "IsIn") 
* Combinaciones de estas mediante las condiciones and, or o not

## <a name="adding-and-transforming-columns"></a>Adición y transformación de columnas

Las siguientes funciones de M agregan o transforman columnas: [Table.AddColumn](/powerquery-m/table-addcolumn), [Table.TransformColumns](/powerquery-m/table-transformcolumns), [Table.ReplaceValue](/powerquery-m/table-replacevalue), [Table.DuplicateColumn](/powerquery-m/table-duplicatecolumn). A continuación se muestran las funciones de transformación admitidas.

* Aritmética numérica
* Concatenación de texto
* Aritmética de fecha y hora (operadores aritméticos, [Date.AddDays](/powerquery-m/date-adddays), [Date.AddMonths](/powerquery-m/date-addmonths), [Date.AddQuarters](/powerquery-m/date-addquarters), [Date.AddWeeks](/powerquery-m/date-addweeks), [Date.AddYears](/powerquery-m/date-addyears))
* Las duraciones se pueden usar para operaciones aritméticas de fecha y hora, pero se deben transformar en otro tipo antes de que se escriban en un receptor (operadores aritméticos, [#duration](/powerquery-m/sharpduration), [Duration.Days](/powerquery-m/duration-days), [Duration.Hours](/powerquery-m/duration-hours), [Duration.Minutes](/powerquery-m/duration-minutes), [Duration.Seconds](/powerquery-m/duration-seconds), [Duration.TotalDays](/powerquery-m/duration-totaldays), [Duration.TotalHours](/powerquery-m/duration-totalhours), [Duration.TotalMinutes](/powerquery-m/duration-totalminutes), [Duration.TotalSeconds](/powerquery-m/duration-totalseconds))    
* La mayoría de las funciones numéricas estándar, científicas y trigonométricas (todas las funciones de [Operaciones](/powerquery-m/number-functions#operations), [Redondeo](/powerquery-m/number-functions#rounding) y [Trigonometría](/powerquery-m/number-functions#trigonometry) *excepto* Number.Factorial, Number.Permutations y Number.Combinations)
* Reemplazo ([Replacer.ReplaceText](/powerquery-m/replacer-replacetext), [Replacer.ReplaceValue](/powerquery-m/replacer-replacevalue), [Text.Replace](/powerquery-m/text-replace), [Text.Remove](/powerquery-m/text-remove))
* Extracción de texto posicional ([Text.PositionOf](/powerquery-m/text-positionof), [Text.Length](/powerquery-m/text-length), [Text.Start](/powerquery-m/text-start), [Text.End](/powerquery-m/text-end), [Text.Middle](/powerquery-m/text-middle), [Text.ReplaceRange](/powerquery-m/text-replacerange), [Text.RemoveRange](/powerquery-m/text-removerange))
* Formato básico de texto ([Text.Lower](/powerquery-m/text-lower), [Text.Upper](/powerquery-m/text-upper), [Text.Trim](/powerquery-m/text-trim)/[Start](/powerquery-m/text-trimstart)/[End](/powerquery-m/text-trimend), [Text.PadStart](/powerquery-m/text-padstart)/[End](/powerquery-m/text-padend), [Text.Reverse](/powerquery-m/text-reverse))
* Funciones de fecha y hora ([Date.Day](/powerquery-m/date-day), [Date.Month](/powerquery-m/date-month), [Date.Year](/powerquery-m/date-year) [Time.Hour](/powerquery-m/time-hour), [Time.Minute](/powerquery-m/time-minute), [Time.Second](/powerquery-m/time-second), [Date.DayOfWeek](/powerquery-m/date-dayofweek), [Date.DayOfYear](/powerquery-m/date-dayofyear), [Date.DaysInMonth](/powerquery-m/date-daysinmonth))
* Expresiones if (pero las ramas deben tener tipos coincidentes)
* Filtros de fila como columna lógica
* Constantes de número, texto, lógica, fecha, y fecha y hora

<a name="mergingjoining-tables"></a>Combinar o unir tablas
----------------------
* Power Query generará una combinación anidada (Table.NestedJoin; los usuarios también pueden escribir manualmente [Table.AddJoinColumn](/powerquery-m/table-addjoincolumn)).
    Los usuarios deben expandir la columna de combinación anidada en una combinación no anidada (Table.ExpandTableColumn no se admite en ningún otro contexto).
* La función de M [Table.join](/powerquery-m/table-join) puede escribirse directamente para evitar la necesidad de un paso adicional de expansión, pero el usuario debe asegurarse de que no haya nombres de columna duplicados entre las tablas combinadas.
* Tipos de combinación admitidos:   [Inner](/powerquery-m/joinkind-inner), [LeftOuter](/powerquery-m/joinkind-leftouter), [RightOuter](/powerquery-m/joinkind-rightouter), [FullOuter](/powerquery-m/joinkind-fullouter)
* Tanto [Value.Equals](/powerquery-m/value-equals) como [Value.NullableEquals](/powerquery-m/value-nullableequals) se admiten como comparadores de igualdad de claves

## <a name="group-by"></a>Agrupar por

Utilice [Table.Group](/powerquery-m/table-group) para agregar valores.
* Debe usarse con una función de agregación
* Funciones de agregación compatibles:   [List.Sum](/powerquery-m/list-sum), [List.Count](/powerquery-m/list-count), [List.Average](/powerquery-m/list-average), [List.Min](/powerquery-m/list-min), [List.Max](/powerquery-m/list-max), [List.StandardDeviation](/powerquery-m/list-standarddeviation), [List.First](/powerquery-m/list-first), [List.Last](/powerquery-m/list-last)

## <a name="sorting"></a>Ordenación

Use [Table.Sort](/powerquery-m/table-sort) para ordenar los valores.

## <a name="reducing-rows"></a>Reducir filas

Mantener y quitar la parte superior, mantener el rango (funciones M correspondientes, solo se admiten recuentos, no condiciones: [Table.FirstN](/powerquery-m/table-firstn), [Table.Skip](/powerquery-m/table-skip), [Table.RemoveFirstN](/powerquery-m/table-removefirstn), [Table.Range](/powerquery-m/table-range), [Table.MinN](/powerquery-m/table-minn), [Table.MaxN](/powerquery-m/table-maxn))

## <a name="known-unsupported-functions"></a>Funciones conocidas no admitidas

| Función | Estado |
| -- | -- |
| Table.PromoteHeaders | No compatible. Se puede lograr el mismo resultado si se establece "Primera fila como encabezado" en el conjunto de resultados. |
| Table.CombineColumns | Se trata de un escenario habitual que no se admite directamente, pero se puede realizar si se agrega una nueva columna que concatene dos columnas concretas.  Por ejemplo, Table.AddColumn(RemoveEmailColumn, "Name", each [FirstName] & " " & [LastName]) |
| Table.TransformColumnTypes | Esto se admite en la mayoría de los casos. No se admiten los siguientes escenarios: transformar una cadena al tipo de moneda, transformar una cadena al tipo de hora, transformar una cadena al tipo de porcentaje. |
| Table.NestedJoin | Si realiza una combinación, se producirá un error de validación. Las columnas deben expandirse para que funcione. |
| Table.Distinct | No se admite la eliminación de filas duplicadas. |
| Table.RemoveLastN | No se admite la eliminación de las filas inferiores. |
| Table.RowCount | No se admite, pero se puede lograr agregando una columna personalizada que contenga el valor 1 y sumando después esa columna con List.Sum. Se admite Table.Group. | 
| Control de errores de nivel de fila | El control de errores de nivel de fila no se admite actualmente. Por ejemplo, para filtrar los valores no numéricos de una columna, una opción sería transformar la columna de texto en números. Cada celda que no se pueda transformar tendrá un estado de error y debe filtrarse. Este escenario no es posible en M con escalabilidad horizontal. |
| Table.Transpose | No compatible |
| Table.Pivot | No compatible |
| Table.SplitColumn | Compatibilidad parcial |

## <a name="m-script-workarounds"></a>Soluciones alternativas de script M

### <a name="for-splitcolumn-there-is-an-alternate-for-split-by-length-and-by-position"></a>Para ```SplitColumn``` hay una alternativa para dividir por longitud y por posición

* Table.AddColumn(Source, "Primeros caracteres", each Text.Start([Email], 7), type text)
* Table.AddColumn(#"Primeros caracteres insertados", "Rango de texto", each Text.Middle([Email], 4, 9), type text)

Se puede acceder a esta opción desde "Extraer" en la cinta de opciones.

![Power Query Agregar columna](media/wrangling-data-flow/pq-split.png)

### <a name="for-tablecombinecolumns"></a>Para ```Table.CombineColumns```

* Table.AddColumn(RemoveEmailColumn, "Nombre", each [FirstName] & " " & [LastName])


## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo [crear una consulta de Power Query de limpieza y transformación de datos en ADF](wrangling-tutorial.md).
