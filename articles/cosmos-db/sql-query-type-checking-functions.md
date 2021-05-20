---
title: Funciones de comprobación de tipos en el lenguaje de consulta de Azure Cosmos DB
description: Obtenga información sobre las funciones de comprobación de tipos del sistema SQL en Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 05/04/2021
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 0a97517e524bbd6f13a108562a85037ae9695187
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108768920"
---
# <a name="type-checking-functions-azure-cosmos-db"></a>Funciones de comprobación de tipos (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Las funciones de comprobación de tipos permiten comprobar el tipo de una expresión dentro de una consulta SQL. Puede usar las funciones de comprobación de tipos para determinar los tipos de propiedades dentro de los elementos sobre la marcha, cuando son variables o desconocidos. 

## <a name="functions"></a>Functions

Las siguientes funciones admiten la comprobación de tipos de valores de entrada y devuelven un valor booleano:

| Función del sistema                           | Uso de índices | [Uso de índices en consultas con funciones de agregado escalares](index-overview.md#index-utilization-for-scalar-aggregate-functions) | Observaciones |
| ----------------------------------------- | ----------- | ------------------------------------------------------------ | ------- |
| [IS_ARRAY](sql-query-is-array.md)         | Examen completo   | Examen completo                                                    |         |
| [IS_BOOL](sql-query-is-bool.md)           | Búsqueda del índice  | Búsqueda del índice                                                   |         |
| [IS_DEFINED](sql-query-is-defined.md)     | Búsqueda del índice  | Búsqueda del índice                                                   |         |
| [IS_NULL](sql-query-is-null.md)           | Búsqueda del índice  | Búsqueda del índice                                                   |         |
| [IS_NUMBER](sql-query-is-number.md)       | Búsqueda del índice  | Búsqueda del índice                                                   |         |
| [IS_OBJECT](sql-query-is-object.md)       | Examen completo   | Examen completo                                                    |         |
| [IS_PRIMITIVE](sql-query-is-primitive.md) | Búsqueda del índice  | Búsqueda del índice                                                   |         |
| [IS_STRING](sql-query-is-string.md)       | Búsqueda del índice  | Búsqueda del índice                                                   |         

## <a name="next-steps"></a>Pasos siguientes

- [Funciones del sistema (Azure Cosmos DB)](sql-query-system-functions.md)
- [Introducción a Azure Cosmos DB](introduction.md)
- [Funciones definidas por el usuario](sql-query-udfs.md)
- [Aggregates](sql-query-aggregate-functions.md) (Agregados)
