---
title: Sintaxis de consulta de búsqueda de Graph
titleSuffix: Azure Machine Learning
description: Aprenda a usar la sintaxis de consulta de búsqueda en el diseñador de Azure Machine Learning para buscar nodos en un grafo de canalización.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 03/24/2021
ms.openlocfilehash: 74cf0b897529e8bb198b6f82a57e187662a4a285
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107259237"
---
# <a name="graph-search-query-syntax"></a>Sintaxis de consulta de búsqueda de Graph

En este artículo aprenderá sobre la funcionalidad de búsqueda en los grafos en Azure Machine Learning. 

La búsqueda en los grafos permite navegar rápidamente por un nodo al depurar o crear una canalización. Puede escribir la palabra clave o la consulta en el cuadro de entrada de la barra de herramientas o en la pestaña de búsqueda del panel izquierdo para desencadenar la búsqueda. Todos los resultados coincidentes se resaltarán en amarillo en el lienzo y, si selecciona un resultado en el panel izquierdo, el nodo del lienzo se resaltará en rojo.

![Captura de pantalla que muestra una experiencia de búsqueda en los grafos de ejemplo.](media/search/graph-search-0322.png)

La búsqueda en gráficos admite la búsqueda de palabras clave de texto completo en los comentarios y el nombre del nodo. También puede filtrar por la propiedad del nodo, como runStatus, duration o computeTarget. La búsqueda de palabras clave se basa en la consulta de Lucene. Una consulta de búsqueda completa tiene el siguiente aspecto:  

**([consulta de lucene] | [consulta por filtro])** 

Puede usar una consulta de Lucene o una consulta por filtro. Para usar ambas opciones, incluya el separador **|** . La sintaxis de la consulta por filtro es más estricta que la consulta de Lucene. Por tanto, si la entrada del cliente se puede analizar con ambas opciones, se aplicará al consulta con filtro.

Por ejemplo, `data OR model | compute in {cpucluster}`, para buscar nodos cuyo nombre o comentario contiene `data` o `model`; el proceso es cpucluster.
 

## <a name="lucene-query"></a>Consulta de Lucene

La búsqueda en gráficos usa la consulta simple de Lucene como sintaxis de búsqueda de texto completo en los "comentarios" y el "nombre" del nodo. Se admiten los siguientes operadores de Lucene:

 
- Y/O
- Coincidencia de caracteres comodín con **?** y operadores **\*** .

### <a name="examples"></a>Ejemplos

- Búsqueda simple: `JSON Data`

- Y/O: `JSON AND Validation`

- Varios Y/O: `(JSON AND Validation) OR (TSV AND Training)`

 
- Coincidencia de caracteres comodín: 
    - `machi?e learning`
    - `mach*ing`
 
>[!NOTE]
> No se puede iniciar una consulta de Lucene con un carácter "*".

##  <a name="filter-query"></a>Consulta con filtro

 
Las consultas con filtro usan el siguiente patrón:
 
`**[key1] [operator1] [value1]; [key2] [operator1] [value2];**`

 
Puede usar las siguientes propiedades de nodo como claves:

- runStatus
- compute
- duration
- reuse
- Publicar
- etiquetas

También puede usar los siguientes operadores:

- Mayor o igual: `>=`
- Menor o igual: `<=`
- Mayor: `>`
- Menor: `<`
- Igual: `==`
- Contiene: `=`
- No igual: `!=`
- En: `in`

 
 

### <a name="example"></a>Ejemplo

- duration > 100;
- status en { Failed,NotStarted}
- compute en {gpu-cluster}; runStatus en {Completed}

## <a name="technical-notes"></a>Notas técnicas

- La relación entre varios filtros es "Y".
- Si se elige `>=,  >,  <, or  <=`, los valores se convertirán automáticamente al tipo numérico. De lo contrario, se usan tipos de cadena para la comparación.
- En el caso de todos los valores de tipo de cadena, no se distinguen mayúsculas de minúsculas en la comparación.
- El operador "In" espera una colección como valor; la sintaxis de la colección es `{name1, name2, name3}`.
- Se omitirá el espacio entre palabras clave.