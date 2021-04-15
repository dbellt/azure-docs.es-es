---
author: brjohnstmsft
ms.service: cognitive-search
ms.topic: include
ms.date: 06/13/2018
ms.author: brjohnst
ms.openlocfilehash: 998d0f1a84dc9cb2a07fb55286c1089787a263e1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "80272899"
---
| Tipo de datos | Características que se permiten en expresiones lambda con `any` | Características que se permiten en expresiones lambda con `all` |
|---|---|---|
| `Collection(Edm.ComplexType)` | Todo excepto `search.ismatch` y `search.ismatchscoring` | Iguales |
| `Collection(Edm.String)` | Comparaciones con `eq` o `search.in` <br/><br/> Combinación de las subexpresiones con `or` | Comparaciones con `ne` o `not search.in()` <br/><br/> Combinación de las subexpresiones con `and` |
| `Collection(Edm.Boolean)` | Comparaciones con `eq` o `ne` | Iguales |
| `Collection(Edm.GeographyPoint)` | Uso de `geo.distance` con `lt` o `le` <br/><br/> `geo.intersects` <br/><br/> Combinación de las subexpresiones con `or` | Uso de `geo.distance` con `gt` o `ge` <br/><br/> `not geo.intersects(...)` <br/><br/> Combinación de las subexpresiones con `and` |
| `Collection(Edm.DateTimeOffset)`, `Collection(Edm.Double)`, `Collection(Edm.Int32)`, `Collection(Edm.Int64)` | Comparaciones mediante `eq`, `ne`, `lt`, `gt`, `le` o `ge` <br/><br/> Combinación de las comparaciones con otras subexpresiones mediante `or` <br/><br/> Combinación de las comparaciones, excepto `ne` con otras subexpresiones mediante `and` <br/><br/> Expresiones que utilizan combinaciones de `and` y `or` en el [formulario normal disyuntivo (DNF)](https://en.wikipedia.org/wiki/Disjunctive_normal_form) | Comparaciones mediante `eq`, `ne`, `lt`, `gt`, `le` o `ge` <br/><br/> Combinación de las comparaciones con otras subexpresiones mediante `and` <br/><br/> Combinación de las comparaciones, excepto `eq` con otras subexpresiones mediante `or` <br/><br/> Expresiones que utilizan combinaciones de `and` y `or` en el [formulario normal conjuntivo (CNF)](https://en.wikipedia.org/wiki/Conjunctive_normal_form) |
