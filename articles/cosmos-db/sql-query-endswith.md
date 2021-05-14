---
title: EndsWith en el lenguaje de consulta de Azure Cosmos DB
description: Obtenga información sobre la función del sistema SQL ENDSWITH en Azure Cosmos DB para devolver un valor booleano que indica si la primera expresión de cadena finaliza con la segunda.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 06/02/2020
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 64514e69b41dda26fc39e747d7fef88706a64c64
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108163884"
---
# <a name="endswith-azure-cosmos-db"></a>ENDSWITH (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Devuelve un valor booleano que indica si la primera expresión de cadena finaliza con la segunda.  
  
## <a name="syntax"></a>Sintaxis
  
```sql
ENDSWITH(<str_expr1>, <str_expr2> [, <bool_expr>])
```  
  
## <a name="arguments"></a>Argumentos
  
*str_expr1*  
   Es una expresión de cadena.  
  
*str_expr2*  
   Es una expresión de cadena que se va a comparar con el final de *str_expr1*.

*bool_expr* Valor opcional para omitir mayúsculas y minúsculas. Cuando se establece en true, ENDSWITH realizará una búsqueda sin distinguir mayúsculas y minúsculas. Si no se especifica, este valor es false.
  
## <a name="return-types"></a>Tipos de valores devueltos
  
  Devuelve una expresión condicional.  
  
## <a name="examples"></a>Ejemplos
  
En el ejemplo siguiente se comprueba si la cadena "abc" termina con "b" y "bC".  
  
```sql
SELECT ENDSWITH("abc", "b", false) AS e1, ENDSWITH("abc", "bC", false) AS e2, ENDSWITH("abc", "bC", true) AS e3
```  
  
 El conjunto de resultados es el siguiente:  
  
```json
[
    {
        "e1": false,
        "e2": false,
        "e3": true
    }
]
```  

## <a name="remarks"></a>Observaciones

Obtenga información sobre [cómo esta función del sistema de cadenas usa el índice](sql-query-string-functions.md).

## <a name="next-steps"></a>Pasos siguientes

- [Funciones de cadena Azure Cosmos DB](sql-query-string-functions.md)
- [Funciones del sistema (Azure Cosmos DB)](sql-query-system-functions.md)
- [Introducción a Azure Cosmos DB](introduction.md)
