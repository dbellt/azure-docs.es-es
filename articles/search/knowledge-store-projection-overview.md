---
title: Conceptos de proyección
titleSuffix: Azure Cognitive Search
description: Dé forma a los datos enriquecidos de la canalización de indexación de enriquecimiento con IA y guárdelos en un almacén de conocimiento para usarlos en escenarios que no sean la búsqueda de texto completo.
manager: nitinme
author: vkurpad
ms.author: vikurpad
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: f2fa631e8ad3da1d40f68c9887b84e4ebd532240
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111559077"
---
# <a name="knowledge-store-projections-in-azure-cognitive-search"></a>"Proyecciones" en un almacén de conocimiento en Azure Cognitive Search

Azure Cognitive Search permite el enriquecimiento de contenido a través de aptitudes cognitivas integradas y aptitudes personalizadas como parte de la indexación. Los enriquecimientos crean información nueva donde no existía anteriormente: extracción de información de imágenes, detección de la opinión, frases clave y entidades del texto, por nombrar algunas. Los enriquecimientos también agregan estructura a texto no diferenciado. Todos estos procesos producen documentos que mejoran la eficacia de la búsqueda de texto completo. En muchos casos, los documentos enriquecidos son útiles para escenarios que no son de búsqueda, como, por ejemplo, para la minería de datos de conocimiento.

Las proyecciones, un componente del [almacén de conocimiento](knowledge-store-concept-intro.md), son vistas de documentos enriquecidos que se pueden guardar en el almacenamiento físico para fines de minería de datos de conocimiento. Una proyección le permite "proyectar" los datos en una forma que se adapte a sus necesidades y manteniendo las relaciones para que herramientas como Power BI puedan leer los datos sin ningún trabajo adicional.

Las proyecciones pueden ser tabulares, con los datos almacenados en filas y columnas en Azure Table Storage, o bien objetos JSON almacenados en Azure Blob Storage. Puede definir varias proyecciones de los datos a medida que se enriquecen. Estas distintas proyecciones resultan útiles cuando quiere que los mismos datos tengan formas diferentes para casos de uso individuales.

El almacén de conocimiento admite tres tipos de proyecciones:

+ **Tablas**: Para los datos que se representan mejor como filas y columnas, las proyecciones de tabla le permiten definir una forma esquematizada o una proyección en el almacenamiento de tablas. Solo los objetos JSON válidos se pueden proyectar como tablas. El documento enriquecido puede contener nodos que no son objetos JSON denominados y, al proyectar estos objetos, crear un objeto JSON válido con una aptitud de conformador o forma en línea.

+ **Objetos**: Cuando necesita una representación JSON de los datos y enriquecimientos, las proyecciones de objeto se guardan como blobs. Solo los objetos JSON válidos se pueden proyectar como objetos. El documento enriquecido puede contener nodos que no son objetos JSON denominados y, al proyectar estos objetos, crear un objeto JSON válido con una aptitud de conformador o forma en línea.

+ **Archivos**: Cuando necesita guardar las imágenes extraídas de los documentos, las proyecciones de archivos le permiten guardar las imágenes normalizadas en el almacenamiento de blobs.

Para ver las proyecciones definidas en contexto, consulte [Creación de un almacén de conocimiento con REST](knowledge-store-create-rest.md).

## <a name="projection-groups"></a>Grupos de proyecciones

En algunos casos, necesitará proyectar sus datos enriquecidos de formas diferentes para satisfacer objetivos diferentes. El almacén de conocimiento le permite definir varios grupos de proyecciones. Los grupos de proyecciones tienen las siguientes características clave de exclusividad mutua y relación.

### <a name="mutual-exclusivity"></a>Exclusividad mutua

Todo el contenido proyectado en un mismo grupo es independiente de los datos proyectados en otros grupos de proyecciones.
Esta independencia implica que puede tener los mismos datos repetidos en cada grupo de proyecciones, pero con una forma diferente.

### <a name="relatedness"></a>Relación

Los grupos de proyecciones ahora permiten proyectar los documentos en todos los tipos de proyección a la vez que se conservan las relaciones entre los tipos de proyección. Todo el contenido proyectado dentro de un mismo grupo de proyecciones mantiene las relaciones entre los datos en los distintos tipos de proyección. Dentro de las tablas, las relaciones se basan en una clave generada y cada nodo secundario mantiene una referencia al nodo primario. En los distintos tipos (tablas, objetos y archivos), las relaciones se conservan cuando un solo nodo se proyecta en varios tipos. Por ejemplo, considere un escenario en el que tiene un documento que contiene imágenes y texto. Podría proyectar el texto en tablas u objetos y las imágenes en archivos donde las tablas u objetos tienen una columna o propiedad que contiene la dirección URL del archivo.

## <a name="input-shaping"></a>Dar forma a los datos de entrada

Dar la forma o la estructura adecuadas a los datos es clave para su uso eficaz, tanto si se trata de tablas como de objetos. La capacidad de modelar o estructurar los datos en función de cómo piensa utilizarlos y acceder a ellos es una capacidad clave denominada como la aptitud de **conformador** en el conjunto de aptitudes.  

Es más fácil definir las proyecciones si tiene un objeto en el árbol de enriquecimiento que coincida con el esquema de la proyección. La [aptitud de conformador](cognitive-search-skill-shaper.md) le permite crear un objeto a partir de diferentes nodos del árbol de enriquecimiento y convertirlos en secundarios de un nuevo nodo. La aptitud de **conformador** le permite definir tipos complejos con objetos anidados.

Cuando haya definido una forma nueva que contiene todos los elementos que necesita para realizar la proyección, podrá usar esta forma como origen para las proyecciones o como entrada para otra aptitud.

## <a name="projection-slicing"></a>Segmentación de proyección

Al definir un grupo de proyecciones, se puede segmentar un solo nodo del árbol de enriquecimiento en varias tablas u objetos relacionados. Agregar una proyección con una ruta de acceso de origen que sea un elemento secundario de una proyección existente hará que el nodo secundario se segmente fuera del nodo primario y se proyecte en la nueva tabla u objeto relacionado. Esta técnica permite definir un único nodo en una aptitud de conformador que puede ser el origen de todas sus proyecciones.

## <a name="table-projections"></a>Proyecciones de tabla

Puesto que facilita la importación, se recomienda el uso de proyecciones de tabla para la exploración de datos con Power BI. Además, las proyecciones de tabla permiten cambiar la cardinalidad entre las relaciones de la tabla. 

Puede proyectar un solo documento del índice en varias tablas, conservando las relaciones. Al proyectar a varias tablas, la forma completa se proyectará en cada tabla, a menos que un nodo secundario sea el origen de otra tabla dentro del mismo grupo.

### <a name="defining-a-table-projection"></a>Definir una proyección de tabla

Al definir una proyección de tabla dentro del elemento `knowledgeStore` del conjunto de aptitudes, empiece asignando un nodo del árbol de enriquecimiento al origen de la tabla. Normalmente, este nodo es el resultado de una aptitud de **conformador** que agregó a la lista de aptitudes para producir una forma concreta que necesita proyectar en las tablas. Puede segmentar el nodo que elija para proyectarlo en varias tablas. La definición de las tablas es una lista de las tablas que desea proyectar.

Cada tabla requiere tres propiedades:

+ tableName: El nombre de la tabla en Azure Storage.

+ generatedKeyName: El nombre de columna para la clave que identifica de forma única esta fila.

+ source: El nodo del árbol de enriquecimiento del que obtiene sus enriquecimientos. Habitualmente, este nodo es el resultado de un conformador, pero podría ser el resultado de cualquiera de las aptitudes.

A continuación se muestra un ejemplo de proyecciones de tabla.

```json
{
    "name": "your-skillset",
    "skills": [
      …your skills
    ],
"cognitiveServices": {
… your cognitive services key info
    },

    "knowledgeStore": {
      "storageConnectionString": "an Azure storage connection string",
      "projections" : [
        {
          "tables": [
            { "tableName": "MainTable", "generatedKeyName": "SomeId", "source": "/document/EnrichedShape" },
            { "tableName": "KeyPhrases", "generatedKeyName": "KeyPhraseId", "source": "/document/EnrichedShape/*/KeyPhrases/*" },
            { "tableName": "Entities", "generatedKeyName": "EntityId", "source": "/document/EnrichedShape/*/Entities/*" }
          ]
        },
        {
          "objects": [ ]
        },
        {
            "files": [ ]
        }
      ]
    }
}
```

Como se muestra en este ejemplo, las entidades y frases clave se modelan en tablas diferentes y contendrán una referencia al elemento primario (MainTable) para cada fila.

## <a name="object-projections"></a>Proyecciones de objeto

Las proyecciones de objeto son representaciones JSON del árbol de enriquecimiento que pueden proceder de cualquier nodo. En muchos casos, la misma aptitud de **conformador** que crea una proyección de tabla se puede usar para generar una proyección de objeto. 

```json
{
    "name": "your-skillset",
    "skills": [
      …your skills
    ],
"cognitiveServices": {
… your cognitive services key info
    },

    "knowledgeStore": {
      "storageConnectionString": "an Azure storage connection string",
      "projections" : [
        {
          "tables": [ ]
        },
        {
          "objects": [
            {
              "storageContainer": "hotelreviews", 
              "source": "/document/hotel"
            }
          ]
        },
        {
            "files": [ ]
        }
      ]
    }
}
```

Para generar una proyección de objeto se requieren algunos atributos específicos del objeto:

+ storageContainer: El contenedor de blobs donde se guardarán los objetos
+ source: La ruta de acceso al nodo del árbol de enriquecimiento que es la raíz de la proyección

## <a name="file-projection"></a>Proyección de archivos

Las proyecciones de archivos son similares a las proyecciones de objetos y solo actúan en la colección de `normalized_images`. De manera similar a las proyecciones de objetos, las proyecciones de archivos se guardan en el contenedor de blobs con el prefijo de carpeta del valor codificado en base64 del identificador de documento. Las proyecciones de archivos no pueden compartir el mismo contenedor que las proyecciones de objetos y deben proyectarse en un contenedor distinto.

```json
{
    "name": "your-skillset",
    "skills": [
      …your skills
    ],
"cognitiveServices": {
… your cognitive services key info
    },

    "knowledgeStore": {
      "storageConnectionString": "an Azure storage connection string",
      "projections" : [
        {
          "tables": [ ]
        },
        {
          "objects": [ ]
        },
        {
            "files": [
                 {
                  "storageContainer": "ReviewImages",
                  "source": "/document/normalized_images/*"
                }
            ]
        }
      ]
    }
}
```

## <a name="projection-lifecycle"></a>Ciclo de vida de las proyecciones

Las proyecciones tienen un ciclo de vida que está asociado a los datos de origen del origen de datos. A medida que los datos se actualizan y se vuelven a indexar, las proyecciones se actualizan con los resultados de los enriquecimientos para garantizar que son coherentes con los datos del origen de datos. Las proyecciones heredan la directiva de eliminación que ha configurado para el índice. Las proyecciones no se eliminan cuando se elimina el indexador o el servicio de búsqueda mismo.

## <a name="using-projections"></a>Uso de las proyecciones

Tras ejecutar el indexador, puede leer los datos proyectados en los contenedores o tablas que especificó a través de las proyecciones.

En cuanto al análisis, la exploración en Power BI es tan sencilla como establecer Azure Table Storage como origen de datos. Puede crear fácilmente un conjunto de visualizaciones en los datos usando las relaciones que contienen.

Como alternativa, si tiene que usar los datos enriquecidos en una canalización de ciencia de datos, podría [cargar los datos desde los blobs en un DataFrame de Pandas](../machine-learning/team-data-science-process/explore-data-blob.md).

Por último, si tiene que exportar los datos desde el almacén de conocimiento, Azure Data Factory tiene conectores para exportar los datos y colocarlos en la base de datos que elija. 

## <a name="next-steps"></a>Pasos siguientes

Como siguiente paso, cree su primer almacén de conocimiento con instrucciones y datos de ejemplo.

> [!div class="nextstepaction"]
> [Creación de un almacén de conocimiento con REST](knowledge-store-create-rest.md)

Para ver un tutorial que abarca conceptos de proyecciones avanzadas como la segmentación, la forma insertada y las relaciones, empiece por [definir proyecciones en un almacén de conocimiento.](knowledge-store-projections-examples.md)

> [!div class="nextstepaction"]
> [Definición de proyecciones en un almacén de conocimiento](knowledge-store-projections-examples.md)
