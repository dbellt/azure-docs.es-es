---
title: Cómo realizar búsquedas en Data Catalog
description: En este artículo se proporciona información general sobre cómo realizar búsquedas en un catálogo de datos.
author: djpmsft
ms.author: daperlov
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 03/16/2021
ms.openlocfilehash: 178604335968c3664bde51c144759c1c040c359d
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107564920"
---
# <a name="search-the-azure-purview-data-catalog"></a>Búsqueda en Azure Purview Data Catalog

Para los consumidores de datos, la detección de datos es el primer paso en una carga de trabajo de análisis o gobernanza de datos. Esta operación puede llevar mucho tiempo, ya que es posible que no sepa dónde encontrar los datos que quiere. Incluso después de encontrarlos, puede que tenga dudas sobre si puede confiar en ellos o adoptarlos.

El objetivo de la búsqueda en Azure Purview es acelerar el proceso de detección de datos, a fin de encontrar rápidamente los datos que realmente importan. En este artículo se describe cómo realizar búsquedas en el catálogo de datos de Azure Purview para encontrar rápidamente los datos que está buscando.

## <a name="search-the-catalog-for-assets"></a>Búsqueda de recursos en el catálogo

En Azure Purview, la barra de búsqueda se encuentra en la parte superior de la experiencia de usuario de Purview Studio.

:::image type="content" source="./media/how-to-search-catalog/purview-search-bar.png" alt-text="Captura de pantalla que muestra la ubicación de la barra de búsqueda de Azure Purview" border="true":::

Al hacer clic en la barra de búsqueda, puede ver el historial de búsquedas recientes y los recursos a los que se ha accedido recientemente. Seleccione "Ver todo" para ver todos los recursos que se han visto recientemente.

:::image type="content" source="./media/how-to-search-catalog/search-no-keywords.png" alt-text="Captura de pantalla que muestra la barra de búsqueda antes de que se hayan escrito las palabras clave" border="true":::

Escriba las palabras clave que ayuden a identificar el recurso, como su nombre, tipo de datos, clasificaciones y términos del glosario. A medida que escribe palabras clave relacionadas con el recurso deseado, Azure Purview muestra sugerencias sobre qué buscar y posibles coincidencias de recursos. Para completar la búsqueda, haga clic en "Ver resultados de la búsqueda" o presione "Entrar".

:::image type="content" source="./media/how-to-search-catalog/search-keywords.png" alt-text="Captura de pantalla que muestra la barra de búsqueda cuando un usuario escribe palabras clave" border="true":::

En la página resultados de la búsqueda se muestra una lista de los recursos que coinciden con las palabras clave que especificó por orden de relevancia. Hay varios factores que pueden afectar a la puntuación de relevancia de un recurso. Puede filtrar más la lista si selecciona almacenes de datos, clasificaciones, contactos, etiquetas y términos del glosario específicos que se apliquen al recurso que está buscando.

:::image type="content" source="./media/how-to-search-catalog/search-results.png" alt-text="Captura de pantalla que muestra los resultados de una búsqueda" border="true":::

 Haga clic en el recurso deseado para ver la página Detalles del recurso, donde podrá ver sus propiedades, incluidos el esquema, el linaje y los propietarios del recurso.

:::image type="content" source="./media/how-to-search-catalog/search-view-asset.png" alt-text="Captura de pantalla que muestra la página de detalles del recurso" border="true":::

## <a name="search-query-syntax"></a>Sintaxis de la consulta de búsqueda

Todas las consultas de búsqueda se componen de palabras clave y operadores. Una palabra clave es un elemento que formaría parte de las propiedades de un recurso. Algunas posibles palabras clave pueden ser una clasificación, un término del glosario, una descripción del recurso o un nombre de recurso. Una palabra clave puede ser solo una parte de la propiedad para la que está buscando una coincidencia. Use las palabras clave y los operadores incluidos a continuación para asegurarse de que Azure Purview devuelve los recursos que está buscando. 

A continuación se muestran los operadores que puede usar para crear una consulta de búsqueda. Los operadores se pueden combinar tantas veces como sea necesario en una misma consulta.

| Operator | Definición | Ejemplo |
| -------- | ---------- | ------- |
| O BIEN | Especifica que un recurso debe tener al menos una de las dos palabras clave. Debe estar en mayúsculas. Un espacio en blanco también es un operador OR.  | La consulta `hive OR database` devuelve los recursos que contienen la palabra "hive" o "database" o ambas. |
| y | Especifica que un recurso debe tener ambas palabras clave. Debe estar en mayúsculas. | La consulta `hive AND database` devuelve los recursos que contienen las palabras "hive" y "database". |
| NOT | Especifica que un recurso no puede contener la palabra clave a la derecha de la cláusula NOT. | La consulta `hive NOT database` devuelve los recursos que contienen la palabra "hive", pero no la palabra "database". |
| () | Agrupa un conjunto de palabras clave y operadores. Al combinar varios operadores, los paréntesis especifican el orden de las operaciones. | La consulta `hive AND (database OR warehouse)` devuelve recursos que contienen la palabra "hive" y la palabra "database" o "warehouse", o ambas. |
| "" | Especifica el contenido exacto de una frase con la que la consulta debe coincidir. | La consulta `"hive database"` devuelve los recursos que contienen la frase "hive database" en sus propiedades. |
| * | Carácter comodín que coincide con uno o varios caracteres; no puede ser el primer carácter de una palabra clave. | La consulta `dat*` devuelve los recursos que tienen propiedades que comienzan por "dat", como "data" o "database". |
| ? | Carácter comodín que coincide con un único carácter. No puede ser el primer carácter de una palabra clave. | La consulta `dat?` devuelve los recursos que tienen propiedades que comienzan por "dat" y son de cuatro letras, como "date" o "data". |

> [!Note]
> Especifique siempre los operadores booleanos de texto (**AND**, **OR**, **NOT**) en mayúsculas. De lo contrario, el caso no es importante, ni tampoco los espacios adicionales.

## <a name="next-steps"></a>Pasos siguientes

- [Cómo crear, importar y exportar términos del glosario](how-to-create-import-export-glossary.md)
- [Cómo administrar plantillas de términos para el glosario empresarial](how-to-manage-term-templates.md)
