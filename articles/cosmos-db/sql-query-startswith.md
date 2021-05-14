---
title: StartsWith con el lenguaje de consulta de Azure Cosmos DB
description: Obtenga información acerca de la función del sistema de SQL STARTSWITH en Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 04/01/2021
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: e3408d64b241e1c22f757f2a076f08693ed8a124
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108162102"
---
# <a name="startswith-azure-cosmos-db"></a>STARTSWITH (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Devuelve un valor booleano que indica si la primera expresión de cadena empieza con la segunda.  
  
## <a name="syntax"></a>Sintaxis
  
```sql
STARTSWITH(<str_expr1>, <str_expr2> [, <bool_expr>])  
```  
  
## <a name="arguments"></a>Argumentos
  
*str_expr1*  
   Es una expresión de cadena.
  
*str_expr2*  
   Es una expresión de cadena que se va a comparar con el principio de *str_expr1*.

*bool_expr* Valor opcional para omitir mayúsculas y minúsculas. Cuando se establece en true, STARTSWITH realizará una búsqueda sin distinguir mayúsculas y minúsculas. Si no se especifica, este valor es false.

## <a name="return-types"></a>Tipos de valores devueltos
  
  Devuelve una expresión condicional.  
  
## <a name="examples"></a>Ejemplos
  
En el ejemplo siguiente se comprueba si la cadena "abc" empieza por "b" y "A".  
  
```sql
SELECT STARTSWITH("abc", "b", false) AS s1, STARTSWITH("abc", "A", false) AS s2, STARTSWITH("abc", "A", true) AS s3
```  
  
 El conjunto de resultados es el siguiente:  
  
```json
[
    {
        "s1": false,
        "s2": false,
        "s3": true
    }
]
```  

## <a name="remarks"></a>Observaciones

Obtenga información [sobre cómo esta función del sistema de cadenas usa el índice](sql-query-string-functions.md).

## <a name="next-steps"></a>Pasos siguientes

- [Funciones de cadena Azure Cosmos DB](sql-query-string-functions.md)
- [Funciones del sistema (Azure Cosmos DB)](sql-query-system-functions.md)
- [Introducción a Azure Cosmos DB](introduction.md)
